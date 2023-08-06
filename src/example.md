# 워크플로우 예시

모델 사이즈가 263.9GiB인 Llama-2-70b 모델을 80GB HBM2E A100이 8장 있는 ferrari 서버에 올리는 예시를 들어겠습니다.

## 퍼시스턴트 볼륨 클레임 생성

먼저 충분한 자원 할당량이 있는지 확인합니다. 아래의 경우는 테스트를 위해 할당량을 매우 크게 잡은 상태입니다.

```sh
$ kubectl get resourcequotas
NAME      AGE   REQUEST
whnbaek   19h   requests.cpu: 0/256, requests.memory: 0/1Ti, requests.nvidia.com/gpu: 0/8, requests.storage: 0/1000Gi
```

모델이 들어가기에 충분하도록 300GB의 퍼시스턴트 볼륨 클레임, `llama-2-70b-pvc.yaml`을 먼저 생성하겠습니다.

```yaml
# llama-2-70b-pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: llama-2-70b-pvc
spec:
  storageClassName: openebs-zfspv
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 300Gi
```

```sh
$ kubectl apply -f llama-2-70b-pvc.yaml
persistentvolumeclaim/llama-2-70b-pvc created
```

## 이미지 빌드

Llama 2는 Pytorch 및 CUDA 환경을 사용합니다. 이들이 미리 세팅된 `nvcr.io/nvidia/pytorch:23.06-py3` 이미지를 베이스로 사용하겠습니다. 이미지는 어느 곳에서 어떠한 방식으로 빌드해도 상관이 없지만 예시에서는 로컬에서 Dockerfile을 작성하여 빌드했습니다. 아래는 빌드에 사용된 Dockerfile입니다.

```dockerfile
# Dockerfile
FROM nvcr.io/nvidia/pytorch:23.06-py3

RUN apt-get update && apt-get install -y wget git

WORKDIR /
RUN git clone https://github.com/facebookresearch/llama.git
RUN cd llama && pip install -e .
```

해당 빌드는 로컬에 보관할 수도 있고, 제공한 Harbor 레지스트리에 보관할 수도 있습니다. Harbor에 올려서 사용하는 시나리오를 예로 들겠습니다.

```sh
$ docker build -t bacchus/llama:latest - < Dockerfile
TODO: harbor image push 하기
```

## 모델 파라미터 다운로드

```yaml
# llama-2-70b-download.yaml
apiVersion: v1
kind: Pod
metadata:
  name: llama-2-70b-download
spec:
  restartPolicy: Never

  containers:
  - name: llama-2-70b-container
    image: "TODO: harbor image 경로"
    command: ["/bin/bash"]
    args: ["-c", "cd /data; echo '$URL_FROM_EMAIL\n$MODELS_TO_DOWNLOAD' | /llama/download.sh;"]
    resources:
      requests:
        cpu: 4
        memory: "1Gi"
    volumeMounts:
    - mountPath: "/data"
      name: llama-2-70b-pv
    env:
    - name: URL_FROM_EMAIL
      value: "the URL from email"
    - name: MODELS_TO_DOWNLOAD
      value: "70B"
  volumes:
  - name: llama-2-70b-pv
    persistentVolumeClaim:
      claimName: llama-2-70b-pvc
```

위 예시에서 `URL_FROM_EMAIL`을 채워넣은 후 실행하여 퍼시스턴트 볼륨에 모델 파라미터를 다운로드합니다.

```sh
$ kubectl apply -f llama-2-70b-download.yaml
pod/llama-2-70b-download created
```

다운로드가 완료되면 완료된 pod을 지우고 실제 실험 컨테이너를 실행합니다.

```sh
$ kubectl delete pods llama-2-70b-download
pod "llama-2-70b-download" deleted
```

## 테스트 실행

```yaml
# llama-2-70b-run.yaml
apiVersion: v1
kind: Pod
metadata:
  name: llama-2-70b-run
spec:
  restartPolicy: Never

  containers:
  - name: llama-2-70b-container
    image: "TODO: harbor image 경로"
    command: ["torchrun"]
    args: ["--nproc_per_node", "8", "/llama/example_text_completion.py", \
        "--ckpt_dir", "llama-2-70b", "--tokenizer_path", "tokenizer.model", \
        "--max_seq_len 128", "--max_batch_size", "4"]
    resources:
      requests:
        cpu: 256
        memory: "1000Gi"
        nvidia.com/gpu: 8
    volumeMounts:
    - mountPath: "/data"
      name: llama-2-70b-pv
  volumes:
  - name: llama-2-70b-pv
    persistentVolumeClaim:
      claimName: llama-2-70b-pvc
```

```sh
$ kubectl apply -f llama-2-70b-run.yaml
pod/llama-2-70b-run created
```

## 그밖에

새로운 소스코드를 작성하거나 데이터셋을 다운로드 받는 경우 경로를 `/data` 내부로 하여 데이터 손실을 방지하는 게 바람직합니다. 새로운 pip 패키지 등을 추가로 설치해야 하는 경우 이미지를 빌드한 후 레지스트리에 저장하는 게 좋습니다. 퍼시스턴트 볼륨으로 파일을 보내거나 꺼내야 하는 경우 `kubectl cp`를 사용하면 됩니다. (`kubectl cp --help` 참고) 다만 이 경우 퍼시스턴트 볼륨이 pod과 붙어 있어야 하므로 무한루프를 걸어 소수의 vCPU와 함께 Pod을 띄운 후 데이터 전송 작업을 할 필요가 있습니다. (작업 완료 후 반드시 Pod 삭제 요망)
