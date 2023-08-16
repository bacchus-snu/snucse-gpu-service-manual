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

해당 빌드는 로컬에 보관할 수도 있고, 제공한 Harbor 레지스트리에 보관할 수도 있습니다. 제공한 레지스트리를 이용할 경우 서버의 로컬 스토리지에 이미지가 저장되므로 컨테이너 실행 시 이미지를 고속으로 pull할 수 있습니다. Harbor에 올려서 사용하는 시나리오를 예로 들겠습니다.

```sh
# https://registry.ferrari.snucse.org:30443 에 bacchus라는 project가 있다고 가정
$ docker build -t registry.ferrari.snucse.org:30443/bacchus/llama:latest - < Dockerfile
$ docker push registry.ferrari.snucse.org:30443/bacchus/llama:latest
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
    image: "registry.ferrari.snucse.org:30443/bacchus/llama:latest"
    command: ["/bin/bash"]
    args: ["-c", "cd /data; echo \"$URL_FROM_EMAIL\n$MODELS_TO_DOWNLOAD\" | /llama/download.sh;"]
    resources:
      limits:
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

위와 같이 데이터셋 및 모델 weight를 준비할 때 소수의 CPU만 할당하여 준비하시기 바랍니다.
스크립트를 한번에 짜서 준비하는 게 어려울 경우

```yaml
    command: [ "/bin/bash", "-c", "--" ]
    args: [ "while true; do sleep 30; done;" ]
```

와 같이 무한루프를 제출하고 `kubectl exec`으로 접속하여 퍼시스턴트 볼륨 내 데이터를 준비할 수 있습니다.
세팅이 끝나면 반드시 해당 pod을 삭제해주시기 바랍니다.

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
    image: "registry.ferrari.snucse.org:30443/bacchus/llama:latest"
    command: ["torchrun"]
    args: ["--nproc_per_node", "8", "/llama/example_text_completion.py", \
        "--ckpt_dir", "llama-2-70b", "--tokenizer_path", "tokenizer.model", \
        "--max_seq_len 128", "--max_batch_size", "4"]
    resources:
      limits:
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

## 기타

소스코드를 수정하거나 패키지를 추가 설치해야 하는 등의 경우 이미지를 다시 빌드한 후 레지스트리에 push해야 합니다. 이미지 빌드 시에는 Dockerfile을 작성하거나 `docker commit`을 이용하면 됩니다. 연구 특성상 소스코드 수정 후 실행을 자주 반복해야 하는 경우가 많습니다. 이때 이미지 레이어가 캐싱되는 것을 고려하지 않은 채 빌드하게 되면 매번 큰 양의 데이터가 네트워크를 통해 전송되게 되므로 수정 후 실행 주기가 길어질 수밖에 없습니다. 이미지 레이어 캐싱을 고려해서 빌드해주시기 바랍니다. 소스코드를 Github에 올린 후 Github Action과 Github Packages를 이용하여 이미지를 자동 빌드하는 방법도 좋습니다. 

퍼시스턴트 볼륨으로 파일을 보내거나 꺼내야 하는 경우 `kubectl cp`를 사용하면 됩니다. (`kubectl cp --help` 참고) 이때 퍼시스턴트 볼륨이 pod과 붙어 있어야 하므로 무한루프를 걸어 소수의 vCPU와 함께 Pod을 띄운 후 데이터를 주고 받으면 됩니다.
