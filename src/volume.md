# 퍼시스턴트 볼륨 설정

컨테이너 내부 스토리지에 쓴 데이터, 로그 파일 등은 ephemeral-storage에 저장됩니다. 이 임시 스토리지는 컨테이너 삭제 시 같이 소멸되므로 데이터셋이나 실행 결과 등을 저장하기에는 부적합합니다. 따라서 SNUCSE GPU 서비스는 퍼시스턴트 볼륨을 추가로 제공합니다. 해당 볼륨은 다음과 같은 예시 `pvc.yaml`로 생성할 수 있습니다.

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx-pvc
spec:
  storageClassName: openebs-zfspv
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 30Gi
```

`storageClassName`은 반드시 `openebs-zfspv`로 설정해야 합니다. 생성은 다음과 같습니다.
```sh
$ kubectl apply -f pvc.yaml
persistentvolumeclaim/nvinx-pvc created
```

퍼시스턴트 볼륨은 컨테이너 종료와 상관 없이 내부가 항상 유지됩니다. 아래는 생성한 퍼시스턴트 볼륨 클레임을 통해 볼륨을 생성하고 이를 `/dataset`에 마운트하는 예시입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  restartPolicy: Never
  containers:
  - name: nginx-container
    image: "nginx:latest"
    command: ["echo"]
    args: ["Hello, world!"]
    resources:
      requests:
        cpu: 2
        memory: "1Gi"
    volumeMounts:
    - mountPath: "/dataset"
      name: nginx-pv
  volumes:
  - name: nginx-pv
    persistentVolumeClaim:
      claimName: nginx-pvc
```

`/dataset` 경로에 데이터셋 및 결과를 저장하면 컨테이너 삭제 후에도 데이터가 보존됩니다.

SNUCSE GPU 서비스 사용 시 추천하는 방식은 다음과 같습니다.
적당한 이미지를 고른 후 필요한 패키지 및 소프트웨어 설치를 합니다. 이미지를 빌드한 후 퍼시스턴트 볼륨을 생성, 소스코드 및 데이터셋을 담습니다. 마지막으로 컨테이너를 제출하여 실행합니다.
