# Terraform 인프라 편입 가이드

기존에 콘솔에서 수동으로 만든 AWS 인프라를 Terraform 관리 대상으로 편입할 때 사용하는 가이드입니다.

이 문서는 다음 목적을 기준으로 작성했습니다.

- Terraform CLI 설치
- AWS 자격 증명 설정
- 기존 운영 리소스 import
- `terraform plan` 으로 현재 코드와 실제 인프라 차이 확인

현재 기준으로는 `EC2`, `보안 그룹` 편입까지만 다룹니다.

---

## 1. Terraform 설치

로컬 PowerShell:

```powershell
winget install HashiCorp.Terraform
```

설치 후 새 PowerShell 열고 확인:

```powershell
terraform -version
```

---

## 2. Terraform 작업 폴더로 이동

```powershell
cd C:\프로젝트 경로\앱 이름\infra\terraform
```

예:

```powershell
cd C:\senaProject\sena\infra\terraform
```

---

## 3. 실제 값 파일 생성

```powershell
Copy-Item terraform.tfvars.example terraform.tfvars
```

- 예시 파일을 실제 값 파일로 복사합니다.
- `terraform.tfvars` 는 실제 운영값이 들어가므로 Git에 올리지 않습니다.

---

## 4. `terraform.tfvars` 에 실제 운영값 입력

- 이 값은 현재 운영 중인 EC2 / 보안 그룹 설정에 따라 달라집니다.
- AWS 콘솔에서 실제 값을 확인해서 입력합니다.

예시 항목:

- `aws_region`
- `instance_name`
- `ami_id`
- `instance_type`
- `key_pair_name`
- `vpc_id`
- `subnet_id`
- `security_group_name`
- `security_group_description`
- `ingress_rules`
- `egress_rules`
- `root_volume_type`
- `root_volume_size`

---

## 5. AWS 자격 증명 준비

### 5-1. AWS CLI 설치 여부 확인

```powershell
aws --version
```

- 버전이 나오면 설치된 상태입니다.
- `명령을 찾을 수 없습니다` 가 나오면 미설치 상태입니다.

### 5-2. AWS CLI 설치

미설치일 때만 진행:

```powershell
winget install Amazon.AWSCLI
```

설치 후 확인:

```powershell
aws --version
terraform -version
```

### 5-3. 현재 자격 증명 유효 여부 확인

```powershell
aws sts get-caller-identity
```

- 계정 정보가 나오면 정상입니다.
- 아래와 같은 에러가 나오면 자격 증명을 다시 설정해야 합니다.

```text
An error occurred (InvalidClientTokenId) when calling the GetCallerIdentity operation: The security token included in the request is invalid
```

### 5-4. 기존 AWS 환경변수 확인

이미 잘못된 인증 정보가 환경변수로 잡혀 있으면 `aws configure` 이후에도 꼬일 수 있습니다.

```powershell
echo $env:AWS_ACCESS_KEY_ID
echo $env:AWS_SECRET_ACCESS_KEY
echo $env:AWS_SESSION_TOKEN
echo $env:AWS_PROFILE
```

- 값이 보이면 현재 세션에 인증 정보가 잡혀 있는 상태입니다.

필요하면 먼저 제거:

```powershell
Remove-Item Env:AWS_ACCESS_KEY_ID -ErrorAction SilentlyContinue
Remove-Item Env:AWS_SECRET_ACCESS_KEY -ErrorAction SilentlyContinue
Remove-Item Env:AWS_SESSION_TOKEN -ErrorAction SilentlyContinue
Remove-Item Env:AWS_PROFILE -ErrorAction SilentlyContinue
```

### 5-5. AWS 자격 증명 설정 방식

둘 중 한 가지 설정만 하면 됩니다.

- `aws configure --profile 프로젝트명`
- 환경변수 직접 설정

기록성과 재사용성을 생각하면 `aws configure --profile 프로젝트명` 방식을 권장합니다.

### 5-6. AWS 콘솔에서 액세스 키 준비

- 개인 프로젝트 기준으로는 보통 IAM 사용자 액세스 키를 사용합니다.
- 루트 계정 액세스 키는 사용하지 않는 것을 권장합니다.

경로:

- `AWS 콘솔 > IAM > 사용자 > 사용자 선택 > 보안 자격 증명 > 액세스 키 만들기`

사용자가 없다면 먼저 생성합니다.

#### IAM 사용자 생성

경로:

- `AWS 콘솔 > IAM > 사용자 > 사용자 생성`

##### 1단계. 사용자 세부 정보 지정

- `사용자 이름` 입력
  - 예: `terraform-user`
  - 프로젝트별로 나누고 싶으면 `terraform-sena` 처럼 작성 가능
- `AWS Management Console에 대한 사용자 액세스 권한 제공` 체크하지 않음
  - Terraform / AWS CLI 용 사용자라 콘솔 로그인은 필요 없습니다.
- `다음` 클릭

##### 2단계. 권한 설정

- 빠르게 실습/개발 환경에서만 확인할 경우 `AdministratorAccess` 로 시작할 수 있습니다.
- 다만 실제 운영 환경에서는 `AdministratorAccess` 사용을 권장하지 않습니다.
- 현재 Terraform 관리 범위가 `EC2`, `보안 그룹`, `EBS`, `EIP` 정도라면 `AmazonEC2FullAccess` 권한으로 시작하는 것이 더 안전합니다.
- 이후 Terraform 관리 범위가 늘어나면 필요한 권한만 추가하는 방식으로 확장합니다.
- `다음` 클릭

##### 3단계. 검토 및 생성

- 설정 내용 확인
- `사용자 생성` 클릭

#### 액세스 키 생성

경로:

- `IAM > 사용자 > 생성한 사용자 클릭 > 보안 자격 증명 탭 > 액세스 키 만들기`

##### 1단계. 사용 사례 선택

- `Command Line Interface(CLI)` 또는 `로컬 코드` 성격의 항목 선택
- 확인 체크 후 `다음`

##### 2단계. 설명 태그 입력

- 선택 사항
- 비워도 됩니다.
- 예: `terraform-local`

##### 3단계. 액세스 키 생성

- 생성 후 아래 두 값을 즉시 저장합니다.
  - `Access Key ID`
  - `Secret Access Key`
- `Secret Access Key` 는 다시 못 보는 경우가 많아서 바로 복사해야 합니다.

### 5-7. 로컬에 자격 증명 등록

추천 방식:

```powershell
aws configure --profile 프로젝트명
```

예:

```powershell
aws configure --profile sena
```

입력 예시:

```text
AWS Access Key ID [None]: AKIA...
AWS Secret Access Key [None]: xxxxxxxxxxxxx
Default region name [None]: ap-northeast-2
Default output format [None]: json
```

### 5-8. Terraform 실행 전 profile 지정

PowerShell:

```powershell
$env:AWS_PROFILE="프로젝트명"
```

예:

```powershell
$env:AWS_PROFILE="sena"
```

정상 등록 확인:

```powershell
aws sts get-caller-identity
```

또는:

```powershell
aws sts get-caller-identity --profile 프로젝트명
```

예:

```powershell
aws sts get-caller-identity --profile sena
```

### 5-9. IDE 터미널에서 profile 지정

기존 IDE가 켜져 있었다면 환경변수 설정이 동기화되지 않을 수 있으므로 IDE 재시작이 필요할 수 있습니다.

Git Bash 기준:

```bash
export AWS_PROFILE=sena
aws sts get-caller-identity
```

---

## 6. Terraform 초기화

현재 프로젝트의 Terraform 작업 폴더에서 provider와 기본 작업 환경을 초기화합니다.

```powershell
terraform init
```

- 처음 실행 시 provider 다운로드가 진행됩니다.
- `.terraform/` 폴더가 생성됩니다.
- 이 폴더는 Git에 올리지 않습니다.

---

## 7. 기존 운영 인프라를 Terraform 관리 대상으로 편입

- 이미 운영 중인 EC2, 보안 그룹을 새로 만드는 것이 아니라 Terraform state에 등록하는 단계입니다.
- 새 인프라를 처음부터 Terraform으로 만드는 경우 이 단계는 생략 가능합니다.
- 현재 운영 환경에서는 `import -> plan` 순서로 가는 것이 안전합니다.

기본 형식:

```powershell
terraform import 리소스명 실제리소스ID
```

예:

```powershell
terraform import aws_security_group.app sg-06f107db56bf2bf1f # 보안 그룹 ID
terraform import aws_instance.app i-0010a5fce97c4a4a6       # 인스턴스 ID
```

설명:

- `aws_security_group.app`
  - `main.tf` 에 정의한 보안 그룹 리소스 이름
- `sg-06f107db56bf2bf1f`
  - AWS 콘솔에서 확인한 실제 보안 그룹 ID
- `aws_instance.app`
  - `main.tf` 에 정의한 EC2 리소스 이름
- `i-0010a5fce97c4a4a6`
  - AWS 콘솔에서 확인한 실제 인스턴스 ID

---

## 8. Terraform 계획 확인

현재 코드와 실제 운영 인프라 차이를 확인합니다.

```powershell
terraform plan
```

이 단계에서 확인할 것:

- 새로 생성될 리소스가 있는지
- 변경될 리소스가 있는지
- 삭제될 리소스가 있는지

기존 운영 인프라를 처음 import 한 직후에는 차이가 조금 보일 수 있습니다.

이 경우:

- `.tf` 코드
- `terraform.tfvars`

를 실제 운영 상태에 맞게 보정합니다.

---

## 9. Terraform state 확인

계획 확인 후 큰 문제가 없으면 여기서부터는 기록 확인 용도입니다.

현재 Terraform이 어떤 리소스를 관리 대상으로 알고 있는지 확인:

```powershell
terraform state list
```

정상이라면 다음 리소스가 보여야 합니다.

- `aws_instance.app`
- `aws_security_group.app`

특정 리소스 상세 확인:

```powershell
terraform state show aws_instance.app
terraform state show aws_security_group.app
```

---

## 10. Terraform 출력값 확인

`outputs.tf` 에 정의한 값 확인:

```powershell
terraform output
```

예:

- 인스턴스 ID
- public ip
- security group id

---

## 11. 이 단계에서는 apply 하지 않음

현재 운영 EC2를 Terraform 관리 대상으로 편입하는 1차 목적은 `import` 와 `plan` 확인입니다.

`terraform apply` 는 `plan` 결과를 충분히 검토한 뒤에만 진행합니다.

```powershell
terraform apply
```

운영 환경에서는 바로 실행하지 말고 먼저 아래를 확인합니다.

- 어떤 값이 달라지는지
- 실제로 수정이 필요한지

---

## 12. 운영 환경에서 주의할 점

- `terraform.tfvars` 는 실제 운영값이 들어가므로 Git에 올리지 않음
- `terraform.tfvars.example` 만 Git에 올림
- `.terraform/`, `*.tfstate`, `crash.log` 는 Git에 올리지 않음
- 운영 환경에서는 `terraform destroy` 사용 금지
- 운영 인프라는 반드시 `import -> plan -> 보정 -> apply` 순서로 진행
