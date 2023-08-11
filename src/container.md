# 컨테이너 실행

## YAML 파일 작성

컨테이너를 실행하기 위해서는 먼저 컨테이너 이미지, 필요한 자원 등을 기술한 YAML 파일을 작성해야 합니다. CPU와 메모리 자원은 필수로 요청해야 하며 자원 할당량 초과 시 실행되지 않습니다. 다음은 nginx를 실행하는 `nginx-pod.yaml` 파일 예시입니다.

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
```

위 예시는 Pod을 기술한 것으로 `nginx:latest` 이미지를 pull하여 `echo "Hello, world!"` 명령어를 수행한 후 종료됩니다. Pod 뿐만 아니라 Deployment, StatefulSet, Job 등 다양한 Kubernetes 워크로드들을 모두 작성하여 실행할 수 있습니다. 자세한 건 사항은 <https://kubernetes.io/docs/concepts/workloads/>를 참고하시기 바랍니다.

## 실행 및 확인

`kubectl apply`를 통해 해당 워크로드를 실행할 수 있습니다. 결과는 `kubectl logs`를 통해 얻을 수 있으며 워크로드의 상태는 `kubectl describe`로 확인할 수 있습니다. 실행이 끝난 워크로드는 `kubectl delete`를 통해 삭제할 수 있습니다.

```sh
$ kubectl apply -f nginx-pod.yaml
pod/nginx-pod created
$ kubectl logs nginx-pod
Hello, world!
$ kubectl describe pods nginx-pod
...
$ kubectl delete pods nginx-pod
pod "nginx-pod" deleted
```

## 주의사항

실행 중인 컨테이너를 아래와 같은 명령어로 접속할 수 있습니다.

```sh
$ kubectl exec nginx-pod -it -- /bin/sh
```

하지만 이를 악용하여, **GPU 자원을 할당한 채 무한 루프를 명령어로 제출한 후 컨테이너에 직접 접속해서 코딩 및 실험을 해서는 안됩니다.** GPU 등의 자원을 제대로 사용하지 않고 독점할 수 있기 때문입니다. 해당 행위는 절대 금물이며 바쿠스는 지속적으로 모니터링을 하여 이를 단속합니다. **Jupyter Notebook와 같은 서버를 띄워두고 port-forwarding을 통해 접속하는 것 역시 금지입니다.** 파이썬 스크립트로 변환 후 제출하는 방식으로 부탁드립니다.

다만, GPU 자원 없이 소수의 vCPU를 할당하여 퍼시스턴트 볼륨을 구성한 후 Pod을 삭제하는 등의 극히 예외적인 경우는 어쩔 수 없는 부분입니다. 자유로운 사용을 위해 많은 기능들을 자유롭게 열어두었으니 여러분의 양심에 맡깁니다. 바람직한 사용법은 워크플로우 예시 부분에서 확인할 수 있습니다.
