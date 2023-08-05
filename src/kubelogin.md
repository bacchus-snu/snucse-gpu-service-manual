# kubelogin 설치 및 설정

## kubelogin 설치

SNUCSE GPU 서비스는 SNUCSE ID <https://id.snucse.org/>와 연동되어 운영됩니다. SNUCSE ID 계정을 통해 서버에 접근하려면 `kubelogin`이라는 `kubectl` 플러그인이 필요합니다. 아래 링크를 참고하여 `kubelogin`을 설치해주시기 바랍니다.

<https://github.com/int128/kubelogin>

`Homebrew`, `Krew`, `Chocolatey`를 통해 설치할 수 있으며, 해당 패키지 매니저가 없을 시 추가 설치가 필요합니다. (`Krew` 설치 매뉴얼 <https://krew.sigs.k8s.io/docs/user-guide/setup/install/>)

## 유저 등록

SNUCSE ID를 서버 접근을 위한 유저로 등록해야 합니다. 명령어는 다음과 같습니다. 유저를 원하는 이름으로 붙일 수 있습니다. (e.g. oidc)

```sh
$ kubectl config set-credentials oidc \
  --exec-api-version=client.authentication.k8s.io/v1beta1 \
  --exec-command=kubectl \
  --exec-arg=oidc-login \
  --exec-arg=get-token \
  --exec-arg=--oidc-issuer-url=https://id-dev.bacchus.io/o \
  --exec-arg=--oidc-client-id=snucse-gpu-service \
  --exec-arg=--oidc-client-secret=snucse-gpu-service
```
