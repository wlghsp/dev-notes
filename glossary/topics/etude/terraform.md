# Terraform

## 한 줄 요약

인프라를 코드로 선언하면, 그 코드와 실제 클라우드 상태를 비교해서 차이만 만들거나 고쳐주는 도구.

---

## 명령형과 선언형의 차이

클라우드 콘솔에서 버튼을 눌러 서버를 만드는 방식은 명령형이다.
"지금 이 순서로 이 작업을 해라"를 사람이 직접 지시하는 방식이기 때문이다.

terraform은 선언형이다.
"인프라가 최종적으로 이런 상태여야 한다"만 `.tf` 파일에 적어두면, terraform이 지금 상태와 비교해서 부족한 걸 만들고 남는 걸 정리한다.
그래서 같은 코드를 몇 번 실행해도 결과가 같다.

---

## state 파일이 하는 일

`.tf` 파일은 설계도이고, `.tfstate` 파일은 그 설계도대로 지금까지 실제로 뭐가 만들어졌는지 기록이다.
terraform은 apply를 실행할 때마다 `.tf`(원하는 상태)와 `.tfstate`(현재 상태)를 비교해서 차이만 실행에 옮긴다.

이 state 파일이 없어지면 terraform은 이미 만든 리소스를 기억하지 못한다.
그래서 다음 apply에서 같은 리소스를 중복으로 만들려고 시도할 수 있다.
etude 프로젝트의 `infra/terraform/terraform.tfstate`가 이 역할을 한다.

---

## 코드 구조 — provider / resource / variable / output

etude의 `infra/terraform/`은 이 네 가지 블록으로 나뉜다.

```hcl
provider "oci" {
  tenancy_ocid = var.tenancy_ocid
  region       = var.region
}
```

provider는 어떤 클라우드에, 어떤 인증으로 접속할지를 정의한다.
etude는 Oracle Cloud(OCI)를 provider로 쓴다.

```hcl
resource "oci_core_vcn" "etude" {
  compartment_id = var.compartment_ocid
  cidr_block     = "10.0.0.0/16"
}
```

resource는 실제로 만들 인프라 하나하나다.
VCN, 서브넷, 방화벽, 서버 인스턴스가 각각 resource 블록 하나씩이다.

```hcl
variable "tenancy_ocid" {}
```

variable은 main.tf 안에서 쓸 값의 이름만 선언한다.
값 자체는 `terraform.tfvars` 파일에 따로 채워 넣는다.
환경(리전, 계정)이 바뀌어도 main.tf는 그대로 두고 tfvars만 바꾸면 되는 이유다.

```hcl
output "public_ip" {
  value = oci_core_public_ip.etude.ip_address
}
```

output은 apply가 끝난 뒤 터미널에 보여줄 값이다.
etude는 서버의 공인 IP와, 바로 쓸 수 있는 ssh 접속 명령어를 output으로 뽑아낸다.

---

## data 블록 — 이미 존재하는 걸 조회만 할 때

```hcl
data "oci_core_images" "ubuntu" {
  operating_system         = "Canonical Ubuntu"
  operating_system_version = "22.04"
  sort_by                  = "TIMECREATED"
  sort_order               = "DESC"
}
```

resource가 새로 만드는 것이라면, data는 클라우드에 이미 있는 것을 조회만 하는 블록이다.
Ubuntu 22.04 이미지의 OCID는 리전마다 다르고 계속 갱신되기 때문에, etude는 이 OCID를 코드에 직접 적지 않고 data로 매번 최신 값을 조회해서 쓴다.

---

## terraform.tfvars에 실제 값이 들어간다

`terraform.tfvars`에는 tenancy OCID, API 키 경로, ssh 공개키 같은 실제 값이 들어 있다.
계정을 식별하는 값이라 git에 커밋되면 안 되고, `.gitignore`로 제외돼 있는지 확인이 필요하다.
