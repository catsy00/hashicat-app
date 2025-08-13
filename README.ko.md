# Hashicat AWS 프로젝트

Hashicat은 AWS에 간단한 "야옹 월드" 웹 애플리케이션을 배포하는 Terraform 프로젝트입니다. HashiCorp 워크숍에서 인프라 프로비저닝, 정책 기반 코드(policy as code)를 위한 Sentinel과의 통합, 기본 애플리케이션 배포 등 Terraform의 기능을 시연하기 위해 설계되었습니다.

이 프로젝트는 VPC, 서브넷, 보안 그룹, 웹 애플리케이션을 제공하는 EC2 인스턴스를 포함하여 완벽하게 독립된 환경을 프로비저닝합니다.

## 사전 요구 사항

시작하기 전에 다음이 설치 및 구성되어 있는지 확인하십시오:

*   **Terraform**: 버전 1.0 이상.
*   **AWS 계정**: Terraform이 사용할 수 있도록 자격 증명이 구성된 활성 AWS 계정. 일반적으로 `AWS_ACCESS_KEY_ID` 및 `AWS_SECRET_ACCESS_KEY` 환경 변수가 설정되어 있어야 합니다.
*   **Git**: 이 리포지토리를 복제하기 위해 필요합니다.

## 사용법

1.  **리포지토리 복제:**
    ```sh
    git clone <repository-url>
    cd hashicat-aws
    ```

2.  **배포 구성:**
    예제 파일을 복사하여 `terraform.tfvars` 파일을 생성합니다:
    ```sh
    cp terraform.tfvars.example terraform.tfvars
    ```
    `terraform.tfvars`를 편집하여 `prefix` 변수에 대한 값을 제공하십시오. 이 접두사는 다른 배포와의 충돌을 피하기 위해 리소스 이름에 사용됩니다. 필요에 따라 이 파일의 다른 변수도 사용자 지정할 수 있습니다.

    ```hcl
    # terraform.tfvars
    prefix = "my-hashicat-app"
    ```

3.  **Terraform 초기화:**
    `terraform init`을 실행하여 필요한 프로바이더를 다운로드합니다.
    ```sh
    terraform init
    ```

4.  **계획 및 적용:**
    `terraform plan`을 실행하여 생성될 리소스를 검토합니다.
    ```sh
    terraform plan
    ```
    계획이 만족스러우면 구성을 적용하여 인프라를 배포합니다.
    ```sh
    terraform apply
    ```
    Terraform은 진행하기 전에 확인을 요청합니다. `yes`를 입력하여 승인합니다.

적용이 완료되면 Terraform은 새 웹 애플리케이션의 URL과 IP 주소를 출력합니다.

## 인프라

이 프로젝트는 다음 AWS 리소스를 생성합니다:

*   `aws_vpc`: 애플리케이션을 위한 전용 가상 프라이빗 클라우드(VPC).
*   `aws_subnet`: VPC 내의 공용 서브넷.
*   `aws_internet_gateway`: 서브넷에 인터넷 액세스를 제공합니다.
*   `aws_route_table`: 서브넷에서 인터넷 게이트웨이로 트래픽을 라우팅합니다.
*   `aws_security_group`: EC2 인스턴스로의 트래픽을 제어하는 방화벽으로, HTTP, HTTPS 및 SSH 액세스를 허용합니다.
*   `aws_instance`: Ubuntu 22.04를 실행하는 EC2 인스턴스 (기본값: `t2.micro`).
*   `aws_eip`: EC2 인스턴스에 정적 공용 IP를 제공하는 탄력적 IP 주소.
*   `aws_key_pair`: 인스턴스에 액세스하기 위한 SSH 키 페어. 개인 키는 `<prefix>-ssh-key.pem`으로 생성 및 저장됩니다.
*   `null_resource`: 프로비저너를 실행하여 Apache를 설치하고 EC2 인스턴스에 웹 애플리케이션을 배포하는 리소스.

## 입력

다음 입력 변수는 `terraform.tfvars` 파일에서 구성할 수 있습니다:

| 이름            | 설명                                                                                             | 유형   | 기본값       | 필수 여부 |
| --------------- | ------------------------------------------------------------------------------------------------------- | ------ | ------------- | :------: |
| `prefix`        | 대부분의 리소스 이름에 포함될 고유한 접두사입니다.                                                 | `string` | -             |   예    |
| `region`        | 리소스가 생성될 AWS 지역입니다.                                                     | `string` | `us-east-1`   |    아니요    |
| `address_space` | VPC에서 사용하는 주소 공간입니다.                                                                             | `string` | `10.0.0.0/16` |    아니요    |
| `subnet_prefix` | 서브넷에 사용할 주소 접두사입니다.                                                                          | `string` | `10.0.10.0/24`|    아니요    |
| `instance_type` | 사용할 EC2 인스턴스 유형입니다.                                                                           | `string` | `t2.micro`    |    아니요    |
| `admin_username`| mysql의 관리자 사용자 이름입니다. (참고: 이 구성에는 MySQL이 설치되어 있지 않습니다).                | `string` | `hashicorp`   |    아니요    |
| `height`        | 자리 표시자 이미지의 높이(픽셀)입니다.                                                          | `string` | `400`         |    아니요    |
| `width`         | 자리 표시자 이미지의 너비(픽셀)입니다.                                                           | `string` | `600`         |    아니요    |
| `placeholder`   | 고양이 이미지에 사용할 이미지 서비스 URL입니다.                                                    | `string` | `placebeard.it` |    아니요    |

## 출력

| 이름         | 설명                                        |
| ------------ | -------------------------------------------------- |
| `catapp_url` | 배포된 웹 애플리케이션의 공용 DNS URL입니다.  |
| `catapp_ip`  | 배포된 웹 애플리케이션의 공용 IP 주소입니다. |

## Sentinel 정책

이 리포지토리에는 `policies/` 디렉토리에 정책 기반 코드(policy-as-code)를 위한 [Sentinel](https://www.hashicorp.com/sentinel) 정책이 포함되어 있습니다. 이 정책들은 Terraform Cloud 또는 Terraform Enterprise와 함께 사용하도록 만들어졌습니다.

*   **`enforce-mandatory-tags`**: 모든 EC2 인스턴스에 `Environment` 및 `Department` 태그가 있도록 요구합니다. `Environment` 태그의 값은 `dev`, `qa` 또는 `prod` 중 하나여야 합니다.
*   **`restrict-deployment-cost`**: 배포에 대한 예상 월간 비용 증가가 $100 미만인지 확인합니다.

이러한 정책은 `soft-mandatory` 적용 수준으로 구성되어 있어 위반 시 경고가 발생하지만 Terraform 실행을 중지하지는 않습니다.

## 연습 문제

`exercises/` 디렉토리에는 가이드 워크숍에서 사용하기 위한 추가 Terraform 파일이 포함되어 있습니다. 이는 주 배포의 일부가 아닙니다.

## 라이선스

이 프로젝트는 [MIT 라이선스](LICENSE)에 따라 라이선스가 부여됩니다.
