# 컨텍스트 설정 및 테스트

## 컨텍스트 등록

앞선 과정을 통해 클러스터와 유저가 등록됐으므로 이 둘을 묶어 컨텍스트로 등록합니다. 이때 네임스페이스를 같이 지정해주어야 하는데, SNUCSE ID와 연동하여 특정 유저가 본인의 유저명에 해당하는 네임스페이스에서만 작업하도록 강제합니다. 따라서 SNUCSE ID 상 유저명을 네임스페이스로 설정합니다. SNUCSE ID 유저명은 [SNUCSE ID](https://id.snucse.org/)에 로그인하여 확인할 수 있습니다. SNUCSE ID 계정이 없으면 가입을 먼저 해주시기 바랍니다. 이전에 `kubectl config set-credentials`에서 등록한 유저 (e.g. `oidc`)와 혼동하지 않도록 주의합니다.

아래는 클러스터가 `bentley`, 유저가 `oidc`, SNUCSE ID 유저명이 `deadbeef`일 때 컨텍스트를 `bentley-oidc`로 등록하는 예시입니다.

```sh
$ kubectl config set-context bentley-oidc \
  --cluster=bentley \
  --user=oidc \
  --namespace=deadbeef
```

## 컨텍스트 변경

여러 서버들을 컨텍스트로 등록하여 이용할 수 있습니다. 이 경우 다음과 같은 방식으로 컨텍스트를 변경할 수 있습니다.

```sh
# ferrari-oidc가 컨텍스트로 등록되어 있을 시 가능
$ kubectl config use-context ferrari-oidc
```

## 테스트

다음 명령어를 통해 서버 접근 가능 여부를 확인할 수 있습니다.

```sh
$ kubectl get namespaces
NAME                     STATUS   AGE
bacchus-gpu-controller   Active   3d2h
cert-manager             Active   3d2h
default                  Active   3d5h
gpu-operator             Active   3d2h
k0s-autopilot            Active   3d5h
kube-node-lease          Active   3d5h
kube-public              Active   3d5h
kube-system              Active   3d5h
openebs                  Active   3d2h
...
```

해당 명령어를 실행할 때 인증을 위해 SNUCSE ID 로그인 창이 켜질 수 있습니다. 로그인 후 권한을 승인하면 됩니다.
서버 접근 과정에서 SNUCSE ID의 그룹 가입 정보를 확인합니다. ferrari 접근 시 학부생 또는 대학원생 그룹에,
bentley 접근 시 대학원생 그룹에 가입되어 있어야 합니다. 그룹 가입 정보가 없거나 잘못된 경우 접근이 거부됩니다.
