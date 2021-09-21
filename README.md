# 목표  
iPaaS와 IoT를 활용해 산업 화재 대응 솔루션을 제작합니다.  

# 아키텍처  

![image](https://user-images.githubusercontent.com/73922068/134149300-e0fb4610-53b4-4cee-947a-acab8a9d87c4.png)

-> Terraform을 통해 AWS의 Route53, EC2, RDS, 보안그룹 및 S3를 생성합니다.  
-> Ansible을 통해 EC2와 RDS에 apache2,php,wordpress,mysql을 설치하고 연결합니다.  

- s3 : .tfstate 파일을 원격 저장소인 S3에 저장하여 파일의 유실을 방지합니다.
- dynamodb : .tfstate 파일을 동시에 작업할 수 없도록 lock을 거는 table 생성  



# 준비사항  
### Terraform  
 - terraform.tfvars  
    - public_key : ec2의 public key  
    - local_ip : ansible을 실행할 컴퓨터의 ip  
### Ansible    
 - private key : ansible playbook을 실행할 때 EC2와의 ssh 연결을 위한 개인키  
 - static.ini : 서버의 ip   

# 작동순서  

1. clone  
``` 
https://github.com/songkiryong/2-tier-wordpress_Terraform.git 
```

2. Terraform  
``` 
terraform init  
terraform apply 
```

3. Ansible    
``` 
ansible-playbook deploy.yaml -b --private-key "~/.ssh/id_rsa" 
```
# 삭제  
1. Terraform  
```
terraform destroy
```
2. Ansible
```
ansible-playbook remove.yaml -b --private-key "~/.ssh/id_rsa"
```

