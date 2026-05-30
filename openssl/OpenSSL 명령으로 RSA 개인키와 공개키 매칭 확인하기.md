# Modulus
`모듈러스`는 RSA 공개 키 알고리즘에서 사용하는 핵심 숫자 값으로 크기가 다른 매우 큰 두 소수의 곱이다. `공개키`와 `개인키`에 모두 포함되어 있으므로 모듈러스 값을 비교하는 것으로 키 쌍이 서로 매칭되는지 확인할 수 있다.

## 키에서 모듈러스 값 확인하기
openssl rsa의 `-modulus` 옵션을 이용하여 키의 모듈러스 값을 확인할 수 있다.

### 개인 키의 모듈러스 값 확인하기
```
openssl rsa -in private.pem -modulus -noout
```
- `openssl rsa`: openssl의 RSA 기능을 이용한다.
- `-in [개인키 파일 경로]`: 개인키 파일 경로를 입력한다. 필자는 'private.pem' 파일로 지정했다.
- `-modulus`: RSA 키 모듈러스를 출력한다.
- `-noout`: 키를 출력하지 않는다.

위 명령을 실행하면 `MODULUS=...`가 표시된다.
### 공개 키의 모듈러스 값 확인하기
```
openssl rsa -pubin -in public.pem -modulus -noout
```
- `-pubin`: 입력 파일로써 공개키를 지정한다.

결과는 위와 마찬가지로 `MODULUS=...`로 표시된다.

이 모듈러스 값이 서로 일치하는지 직접 눈으로 확인할 수 있다.

### 두 명령을 한 번에 실행하는 스크립트
```
(diff <(openssl rsa -in [비밀키 경로] -modulus -noout) <(openssl rsa -pubin -in [공개키 경로] -modulus -noout)) > /dev/null && echo "match" || echo "not match"
```
**주의**: `<()` (프로세스 치환)은 일부 쉘에서 작동하지 않을 수 있다.

결과
- 두 키가 서로 쌍인 경우: `match`
- 두 키가 서로 쌍이 아닌 경우: `not match`

`diff` 명령을 사용한다. diff 명령은 두 내용이 일치하면 아무런 출력도 하지 않고(return 0), 일치하지 않으면 일치하지 않는 부분을 출력하는 명령어이다. (return 1)

따라서 성공적으로 실행되었다면 return 0, "match"를 출력하고, 그렇지 않으면 "not match"를 출력하게 된다.

# 부록
모듈러스 값을 직접 확인하기에는 내용이 긴데, 이를 `md5`로 해시하여 짧게 해시된 값을 확인하면 보기 편하다.
``` bash
$ openssl rsa -in private.pem -modulus -noout | md5sum

$ openssl rsa -pubin -in public.pem -modulus -noout | md5sum
```

### 주의사항
확인한 값은 `MODULUS=...`와 같이 'MODULUS=' 문자열을 포함한 값이기 때문에, 실제 모듈러스 값을 저장하거나 해시가 필요한 경우에는 반드시 해당 문자열을 제거하여야 한다.
