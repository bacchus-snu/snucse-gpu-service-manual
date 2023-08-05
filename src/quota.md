# 자원 할당량 설정

## 부트스트랩 생성

서버를 접근할 수 있다고 하더라도 유저에게 자원 할당량이 부여되지 않아서 작업을 제출, 실행할 수 없습니다. 자원 할당량을 부여 받으려면 먼저 유저 부트스트랩을 작성해야 합니다. SNUCSE ID 유저 이름이 bacchus라고 한다면, `ub.yaml`의 양식은 아래와 같습니다.

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


## 할당량 신청

할당될 자원 목록은 다음과 같습니다. (ephemeral-storage 제한은 추후 지원 예정)
- cpu
- memory
- gpu
- storage

아래 구글 폼을 통해 필요한 자원 할당량을 작성하여 제출합니다.

<https://docs.google.com/forms/u/2/d/1NZnb3B38pVA4CHbN2W78EuJoW5TXQC_hRKYCzPDtEOc/edit?usp=drive_web>

바쿠스에서 검토한 후 승인 또는 거부를 하게 됩니다. 승인될 시 자원 할당량이 부여되며 아래 명령어를 통해 확인할 수 있습니다.

```sh
$ kubectl get resourcequotas
NAME      AGE     REQUEST                LIMIT
bacchus   4h41m   requests.cpu: 0/4, ...
```

자원 할당량 변경이 필요할 경우 구글 폼을 통해 다시 신청할 수 있습니다.
