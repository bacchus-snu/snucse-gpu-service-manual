# 컨텍스트 설정 및 테스트

## 컨텍스트 등록

앞선 과정을 통해 클러스터와 유저가 등록됐으므로 이 둘을 묶어 컨텍스트로 등록합니다. 이때 네임스페이스는 SNUCSE ID 유저 이름을 사용합니다. SNUCSE ID 유저 이름은 <https://id.snucse.org/>에 로그인하여 확인할 수 있습니다. SNUCSE ID 계정이 없으면 가입을 먼저 해주시기 바랍니다. 이전에 `kubectl config set-credentials`로 등록한 유저 이름과 혼동하지 않도록 주의해야 합니다.

아래는 클러스터 이름이 ferrari, 유저 이름이 oidc, SNUCSE ID 유저명이 bacchus일 때의 예시입니다.

```sh
$ kubectl config set-context ferrari-oidc \
  --cluster=ferrari \
  --user=oidc \
  --namespace=bacchus
```

## 컨텍스트 변경

여러 서버들을 컨텍스트로 등록하여 이용할 수 있습니다. 이 경우 다음과 같은 방식으로 컨텍스트를 변경할 수 있습니다.

```sh
$ kubectl config use-context bentley-oidc  # bentley-oidc가 컨텍스트로 등록되어 있을 시
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