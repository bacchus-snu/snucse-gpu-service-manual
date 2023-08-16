# 이미지 레지스트리 이용

사용하실 개인 이미지 레지스트리가 없으신 유저분들을 위해서 Bacchus는 레지스트리를 각 GPU 노드에서 제공합니다.

## 주의사항

현재 레지스트리에 대해서 만드실 수 있는 레지스트리의 개수를 제한하고 있지는 않습니다. 하지만 다른 이용자들도 함께 이용하는 서비스이므로 한 유저 당 1개의 레지스트리만 만들어서 사용하시길 바랍니다. 각 레지스트리는 기본적으로 30GB의 스토리지 Quota를 가지고 있습니다.

만약 어떤 유저가 레지스트리를 무분별하게 만들어서 다른 유저들이 스토리지 공간이 부족해지는 경우가 생기게 된다면, 그 유저의 레지스트리는 예고 없이 삭제될 수 있고, 향후 GPU 서비스의 이용이 제한될 예정입니다.


## 제공 중인 레지스트리

현재 제공되고 있는 레지스트리는 다음과 같습니다.

* [ferrari](https://registry.ferrari.snucse.org:30443/)
* [bentley](https://registry.bentley.snucse.org:30443/)


## 이용 방법

### 프로젝트 생성

1. 이용하고 싶은 레지스트리의 URL에 접속하고, `LOGIN VIA OIDC PROVIDER` 버튼을 통해서 로그인합니다.
1. 좌측의 `Projects` 탭으로 이동하고, `New Project` 버튼을 눌러서 새로운 프로젝트를 생성합니다.

### 레지스트리 로그인

기본적으로 다른 이미지 레지스트리를 사용하는 방법과 같습니다. 이 경우 CLI에서 로그인을 해야하는데, 이 때 사용되는 Credential은 다음과 같습니다.

* `Username`: ID username
* `Password`: CLI secret

`CLI secret`을 얻기 위해서는 다음을 따릅니다.

1. 우측 상단의 버튼에서 `User Profile`을 누릅니다.
1. CLI Secret 란의 오른쪽에 있는 Copy 버튼을 눌러서 클립보드에 시크릿을 복사합니다.

이후, CLI에서 다음과 같이 로그인합니다.

```sh
# 사용할 레지스트리에 따라서 URL은 바뀌게 됩니다. 여기서는 ferrari의 레지스트리를 예시로 들겠습니다.
docker login https://registry.ferrari.snucse.org:30443/

# 이후 prompt에서 물어보는 username과 password를 입력합니다.
```
