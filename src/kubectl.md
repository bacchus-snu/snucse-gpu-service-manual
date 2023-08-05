# kubectl 설치 및 설정

## kubectl 설치

SNUCSE GPU 서비스는 Kubernetes 클러스터를 활용합니다. 따라서 서버에 접근하기 위해서는 `kubectl` 커맨드라인 툴이 필요합니다. 아래 링크를 참고하여 `kubectl`을 설치해주시기 바랍니다.

<https://kubernetes.io/docs/tasks/tools/#kubectl>

## 클러스터 등록

`kubectl`을 통해서 서버의 엔트리포인트와 인증서를 등록해야합니다.

| 서버명   | 엔트리포인트                 | 인증서                                  |
|:------- |:-------------------------- |:-------------------------------------- |
| ferrari | https://147.46.15.75:6443  | [ferrari.crt](./materials/ferrari.crt) | 
| bentley | https://147.46.92.213:6443 | [bentley.crt](./materials/bentley.crt) |

해당하는 인증서를 적절한 경로에 다운로드한 후 (e.g. `~/.kube/`) 예시와 같은 명령어를 통해 클러스터를 등록합니다. 클러스터를 원하는 이름으로 붙일 수 있습니다. (e.g. ferrari)
```sh
# ferrari 서버 등록 예시, 인증서 경로: ~/.kube/ferrari.crt
$ kubectl config set-cluster ferrari \
  --server=https://147.46.15.75:6443 \
  --certificate-authority=ferrari.crt
```
