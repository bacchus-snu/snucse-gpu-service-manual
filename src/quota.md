# 리소스 쿼터 설정

## 부트스트랩 생성

서버를 접근할 수 있다고 하더라도 유저에게 리소스 쿼터가 부여되지 않아서 작업을 제출, 실행할 수 없습니다. 리소스 쿼터를 받으려면 먼저 작업할 서버에서 유저 부트스트랩을 작성해야 합니다. SNUCSE ID 유저 이름이 `bacchus`라고 한다면, `ub.yaml`의 양식은 아래와 같습니다.

```yaml
# ub.yaml
apiVersion: bacchus.io/v1
kind: UserBootstrap
metadata:
  name: bacchus
spec: {}
```

해당 파일을 작성한 후 다음 명령어를 실행합니다.

```sh
$ kubectl apply -f ub.yaml
```

본인의 부트스트랩이 생성됐는지를 확인합니다.
```sh
$ kubectl get userbootstraps.bacchus.io
NAME      AGE
bacchus   4h37m
```


## 리소스 쿼터 신청

리소스 종류는 다음과 같습니다. (ephemeral-storage 추후 제한 예정)
- cpu
- memory
- gpu
- storage

아래 구글 폼을 통해 필요한 리소스 쿼터 요청을 작성하여 제출합니다.


<https://docs.google.com/forms/d/e/1FAIpQLSeC3KTu0ofDaSUTZEJKsEGjTLMgupENkLxR9aVQ2LanA1Spaw/viewform?usp=sf_link>

바쿠스에서 검토 후 승인 또는 거부를 하게 됩니다. 승인될 시 리소스 쿼터가 부여되며 아래 명령어를 통해 확인할 수 있습니다.

```sh
$ kubectl get resourcequotas
NAME      AGE     REQUEST                LIMIT
bacchus   4h41m   requests.cpu: 0/4, ...
```

리소스 쿼터 변경이 필요할 경우 구글 폼을 통해 다시 신청할 수 있으며, 여러 제출 중 승인된 가장 마지막 제출을 기준으로 리소스 쿼터가 설정됩니다.
