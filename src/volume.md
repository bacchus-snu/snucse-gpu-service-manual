# 퍼시스턴트 볼륨 설정

컨테이너 내부 스토리지에 쓴 데이터, 로그 파일 등은 ephemeral-storage에 저장됩니다. 이 임시 스토리지는 컨테이너 삭제 시 같이 소멸되므로 데이터셋이나 실행 결과 등을 저장하기에는 부적합합니다. 따라서 SNUCSE GPU 서비스는 ZFS 기반의 퍼시스턴트 볼륨을 추가로 제공합니다. 여러분은 퍼시스턴트 볼륨 클레임을 통해서 이 퍼시스턴트 볼륨의 일부를 사용할수 있습니다. 다음 예시 `nginx-pvc.yaml`는 30GiB의 퍼시스턴트 볼륨 클레임을 생성하는 예시입니다.

```yaml
# nginx-pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 30Gi
```

```sh
$ kubectl apply -f nginx-pvc.yaml
persistentvolumeclaim/nvinx-pvc created
```

아래는 생성한 퍼시스턴트 볼륨 클레임을 통해 볼륨을 `/dataset`에 마운트하는 예시입니다.

```yaml
# nginx-pod.yaml
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
