Vũ Mạnh Hùng
vmhung290791@gmail.com
- [Nguồn](https://www.amazon.com/Implementing-Cloud-Design-Patterns-AWS-ebook/dp/B00WX3W43I)

- Đang update phần **Snapshoot Patterns**

# GHI CHÉP QUÁ TRÌNH ĐỌC VÀ THỰC HÀNH CUỐN Implement_Cloud_Design_Patterns
## MỤC LỤC
- [Giới thiệu về Vagrant](https://github.com/hungran/AWS-ghichep-implementing-cloud-design-patterns#gi%E1%BB%9Bi-thi%E1%BB%87u-v%E1%BB%81-vagrant) / Công cụ được nhắc đến trong cuốn
- [Snapshoot patterns](https://github.com/hungran/AWS-ghichep-implementing-cloud-design-patterns#snapshoot-patterns) / Thực hành tạo snapshoot / Nâng cao bằng snapshot lifecycle để tự động hóa quy trình snapshoot

*Môi trường yêu cầu*:

	- Tài khoản AWS (khuyên dùng tài khoản free)
	- Ubuntu 16.04 LTS freetier, t2.micro
## Lưu ý: Nhớ tắt hoặc xóa các resource sau khi lab để tránh mất chi phí

### Giới thiệu về Vagrant
- Là công cụ xây dựng, quản lý máy ảo, có thể chạy trên Ubuntu, macOS và Windows
- Ngôn ngữ được sử dụng là `ruby`
- Máy ảo có thể chạy trên các môi trường từ on-prem ví dụ như `Vmware`, `HyperV(?)`, hoặc cloud `AWS` ...
- Bài viết & hướng dẫn cài đặt trên Windows (tiếng Việt) [Vagrant-viblo.asia](https://viblo.asia/p/tim-hieu-vagrant-phan-1-1l0rvmDQGyqA)
- Hướng dẫn cài đặt plug in aws cho vagrant [Link chính thức](https://github.com/mitchellh/vagrant-aws)
- *Lưu ý: dùng cách sau để fix bug khi chạy windows 10 pro 64bit*
	`The "libxml2" package isn't available #539`
	- Hướng dẫn:
		- 1. Install fog-ovirt 1.0.1
			`vagrant plugin install --plugin-version 1.0.1 fog-ovirt`
		- 2. Install vagrant-aws
			`vagrant plugin install vagrant-aws`
	- Reason: fog-ovirt is one of the dependencies and since version 1.0.2 it depends on ovirt-engine-sdk which is giving trouble
## Đồng bộ thư mục giữa máy host và EC2 bằng rsync
---
- Cài thư viện Cygwin64 từ [link](https://cygwin.com/install.html) và làm theo hướng dẫn sau [link](https://site.elastichosts.com/blog/installing-cygwin-on-windows-for-linux-tools/) **Lưu ý: chọn các library rsync theo hướng dẫn**


- Tham khảo [Vagrantfile](https://github.com/hungran/AWS-ghichep-implementing-cloud-design-patterns/blob/master/Vagrantfile) sample ở đây :)

	- Một số lệnh cơ bản của vagrant:
		1. `vagrant up --provider=aws` chạy vagrant với aws từ `vagrantfile`
		2. `vagrant reload --provision` khởi động lại machine\instance\vm
		3. `aws.user_data` dùng để định nghĩa bootstrap thay vì dùng [provision](https://www.vagrantup.com/intro/getting-started/provisioning.html)
		4. `vagrant destroy` terminate machine\instance\vm
		5. `vagrant halt --force` shutdown machine\instance\vm
		6. `vagrant reload --provision` khởi động lại machine\instance\vm từ `vagrantfile` đã dược thay đổi
		7. `vagrant ssh` ssh trực tiếp vào machine\instance\vm vừa khởi chạy
		8. `vagrant rsync` đồng bộ lại thư mục giữa host và machine\instance\vm đang chạy

- Tạo EC2 cho lab Snapshoot patterns bằng `vagrantfile` & `bootstrap` có nội dung như dưới:
		
		Vagrantfile:
		---
			Vagrant.configure("2") do |config|
			config.vm.box = "dummy"
			config.vm.provider :aws do |aws, override|
		# 	Using rsync by cygwin64 instead SMB share
			config.vm.allowed_synced_folder_types = [:rsync]	
			aws.access_key_id = File.read("access_key.txt")
			aws.secret_access_key = File.read("secret_access_key.txt")
		#	aws.session_token = "SESSION TOKEN"
		#	aws.keypair_name = "KEYPAIR NAME"
			aws.region = "ap-southeast-1"
		#	ubuntu 16.04 LTS
			aws.ami = "ami-0ee0b284267ea6cde"
			aws.instance_type = "t2.micro"
			aws.availability_zone = "ap-southeast-1b"
			aws.subnet_id = "subnet-04fa887456cd2ab68"
			aws.associate_public_ip = true
			aws.private_ip_address = "172.31.16.50"
			aws.security_groups = "sg-04625159d166466da"
			aws.tags = {
				'Name' => 'Hung_Test_Vagrantfile_Bootstrap_02'
			}
			aws.keypair_name = "hung20190616"
			aws.user_data = File.read("bootstrap.txt")
		#	Câu lệnh mặc định sử dụng user name
			override.ssh.username = "ubuntu"
		#	Câu lệnh mặc định sử dụng keypem dùng để ssh đến instance	
			override.ssh.private_key_path = "C:/HashiCorp/hung20190616.pem"
			
		  end
		end
		
		Bootstrap: 
		---
		
		#! /bin/bash
		sudo apt-get upgrade
		sudo apt-get update -y
		sudo apt install awscli -y
		sudo apt-get install apache2 -y
		sudo service apache2 start
		sudo echo "public ip is $(curl http://169.254.169.254/latest/meta-data/public-ipv4), " >> hung.txt	
		sudo echo "instance id is $(curl  http://169.254.169.254/latest/meta-data/instance-id)," >> hung.txt
		sudo echo "instance-type is $(curl  http://169.254.169.254/latest/meta-data/instance-type)." >> hung.txt
		sudo cat hung.txt > /var/www/html/index.html
		
- Ý nghĩa: Cài đặt ec2 từ `AMI ubuntu 16.04 LTS` và chạy web `apache2`, **lấy [metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html)** của instance đẩy vào file **"index.html"** **Script ngáo đá, mong người đọc thông cãm :)**

- Kết quả:

<img src ="https://imgur.com/Gm0ZqDO.jpg">

- Kiểm tra sync giữa host & instance bằng cách ssh vào instance qua lệnh `vagrant ssh`

- Kết quả:

<img src =" ">

## Snapshoot patterns
### Tạo snapshoot `ebs` running instance
- 1. Khởi chạy ec2 instance từ vagrantfile bằng lệnh `vagrant up --provider=aws`
- 2. Vào giao diện GUI console -> tìm đến EC2 -> chọn instance đang chạy, tìm đến phần **Description** -> trỏ chuột và chọn đến link **volume-id** có dạng **vol-xxx**
	
<img src ="https://imgur.com/dPORWBp.jpg">
	
- 3. Tại giao diện Volumes section, chuột phải chọn Volume cần snapshoot chọn **Create Snapshot**
		
<img src ="https://imgur.com/iLGIEo3.jpg">
	
- 4. Giao diện **Create Snapshot** popup ra, ta điền các tham số như hình:	
		
<img src ="https://imgur.com/M2yB2C4.jpg">
	
- 5. Kết quả:
	
<img src ="https://imgur.com/HJ6cCnW.jpg">	
### NEW!!! Nâng cao Dùng snapshot lifecycle tự động tạo và xóa snapshoot
Miệt mài commit miệt mài push --> vận may đến?
