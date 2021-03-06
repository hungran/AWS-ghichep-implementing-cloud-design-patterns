# UPDATED: BUG with new Vagrant version 2.2.7 (release on Jan 27 2020)
FIXED:

- Vagrant 2.2.6 [LINK](https://releases.hashicorp.com/vagrant/2.2.6/)

- Install plugin fog-ovirt 1.0.1:

	`vagrant plugin install --plugin-version 1.0.1 fog-ovirt`

- Install plugin vagrant-aws:

	`vagrant plugin install vagrant-aws`
	
Vũ Mạnh Hùng
vmhung290791@gmail.com
- [Nguồn](https://www.amazon.com/Implementing-Cloud-Design-Patterns-AWS-ebook/dp/B00WX3W43I)

- Đang update phần **Stamp pattern**

# GHI CHÉP QUÁ TRÌNH ĐỌC VÀ THỰC HÀNH CUỐN Implement_Cloud_Design_Patterns
## MỤC LỤC
- [Giới thiệu về Vagrant](https://github.com/hungran/AWS-ghichep-implementing-cloud-design-patterns#gi%E1%BB%9Bi-thi%E1%BB%87u-v%E1%BB%81-vagrant) / Công cụ được nhắc đến trong cuốn
- [snapshot patterns](https://github.com/hungran/AWS-ghichep-implementing-cloud-design-patterns#snapshot-patterns) / Thực hành tạo snapshot:
	- [Appendix_NEW](https://github.com/hungran/AWS-ghichep-implementing-cloud-design-patterns#appendix_new-n%C3%A2ng-cao-d%C3%B9ng-snapshot-lifecycle-t%E1%BB%B1-%C4%91%E1%BB%99ng-t%E1%BA%A1o-v%C3%A0-x%C3%B3a-snapshot) / Nâng cao bằng snapshot lifecycle để tự động hóa quy trình snapshot
- [Stamp pattern]() / Updating / Sử dụng `vagrant` để khởi chạy ec2 phục vụ lab / fix lỗi không `sudo` từ AMI Linux để rsycn thư mục (có thể bỏ qua nếu sử dụng AMI Ubuntu)/ tạo AMI từ ec2 này
- [ScaleUp Patterns]() / ScaleUp bằng GUI

*Môi trường yêu cầu*:

	- Tài khoản AWS (khuyên dùng tài khoản free)
	- Ubuntu 16.04 LTS freetier, t2.micro
	- Vagrant 2.2.6 

## Lưu ý: Nhớ tắt hoặc xóa các resource sau khi lab để tránh mất chi phí

### Giới thiệu về Vagrant
- Là công cụ xây dựng, quản lý máy ảo, có thể chạy trên Ubuntu, macOS và Windows
- Ngôn ngữ được sử dụng là `ruby`
- Máy ảo có thể chạy trên các môi trường từ on-prem ví dụ như `Vmware`, `HyperV(?)`, hoặc cloud `AWS` ...
- Bài viết & hướng dẫn cài đặt trên Windows (tiếng Việt) [Vagrant-viblo.asia](https://viblo.asia/p/tim-hieu-vagrant-phan-1-1l0rvmDQGyqA)
- Link download phiên bản Vagrant 2.2.6 [Link](https://releases.hashicorp.com/vagrant/2.2.6/)
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
	6. `vagrant reload --provision` khởi động lại machine\instance\vm từ `vagrantfile` đã được thay đổi
	7. `vagrant ssh` ssh trực tiếp vào machine\instance\vm vừa khởi chạy
	8. `vagrant rsync` đồng bộ lại thư mục giữa host và machine\instance\vm đang chạy
	
- Tạo EC2 cho lab snapshot patterns bằng `vagrantfile` & `bootstrap` có nội dung như dưới:
		
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
		sudo apt-get install awscli -y
		sudo apt-get install apache2 stress -y
		sudo service apache2 start
		sudo echo "public ip is $(curl http://169.254.169.254/latest/meta-data/public-ipv4), " >> hung.txt	
		sudo echo "instance id is $(curl  http://169.254.169.254/latest/meta-data/instance-id)," >> hung.txt
		sudo echo "instance-type is $(curl  http://169.254.169.254/latest/meta-data/instance-type)." >> hung.txt
		sudo cat hung.txt > /var/www/html/index.html
		
- Ý nghĩa: Cài đặt ec2 từ `AMI ubuntu 16.04 LTS` và chạy web `apache2`, **lấy [metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html)** của instance đẩy vào file **"index.html"** **Script ngáo đá, mong người đọc thông cãm :)**

- Kết quả:

<img src ="https://imgur.com/Gm0ZqDO.jpg">

- Kiểm tra sync giữa host & instance bằng cách ssh vào instance qua lệnh `vagrant ssh`

## snapshot patterns
### Tạo snapshot `ebs` running instance
- 1. Khởi chạy ec2 instance từ vagrantfile bằng lệnh `vagrant up --provider=aws`
- 2. Vào giao diện GUI console -> tìm đến EC2 -> chọn instance đang chạy, tìm đến phần **Description** -> trỏ chuột và chọn đến link **volume-id** có dạng **vol-xxx**
	
<img src ="https://imgur.com/dPORWBp.jpg">
	
- 3. Tại giao diện Volumes section, chuột phải chọn Volume cần snapshot chọn **Create Snapshot**
		
<img src ="https://imgur.com/iLGIEo3.jpg">
	
- 4. Giao diện **Create Snapshot** popup ra, ta điền các tham số như hình:	
		
<img src ="https://imgur.com/M2yB2C4.jpg">
	
- 5. Kết quả:
	
<img src ="https://imgur.com/HJ6cCnW.jpg">

### Appendix_NEW!!! Nâng cao Dùng snapshot lifecycle tự động tạo và xóa snapshot
- 1. Khởi chạy ec2 instance từ `vagrantfile` bằng lệnh `vagrant up --provider=aws`
- 2. Vào giao diện GUI console -> tìm đến EC2 -> tìm đến tab **lifecycle policy manager** -> **Create Snapshot Lifecycle Policy** ta điền tham số như hình

<img src ="https://imgur.com/62c4ksN.jpg">

- Lưu ý: Target chọn đến tag như tag `Name` value `Hung_Test_Vagrantfile_Bootstrap_02` như `vagrantfile` ta định nghĩa
- Đánh tags cho chính policy này:
		
		Key: Name
		Value: snapshot_policy
	
- Đặt lịch -> Tên lịch
- Run policy every *Amazon chỉ cho các lựa chọn chạy 2, 3, 4, 6, 8, 12, 24 giờ.* Ở đây ta chọn 24 giờ
- Starting at hh:mm UTC
- **Retention** có 2 dạng **Count** & **Age** ở đây ta chọn **Count**
- Do chọn **Count** nên sẽ có lựa chọn **Retain**, ta để 7 
- Kết quả:
		
	<img src ="https://imgur.com/wCdTGkO.jpg">
	
- Có các lựa chọn **Cross region copy(optional)**
- IAM Role mặc định Amazon sẽ đặt Default role cho Snapshot này, default role sẽ cho phép ec2 tạo, sửa, xóa, view, describe snapshot:
		
		ARN của IAM default role cho snapshot: arn:aws:iam::xxxxxx:role/service-role/AWSDataLifecycleManagerDefaultRole
		
- Có lựa chọn để **enable** hoặc **disable** policy này khi khởi tạo
		
	<img src ="https://imgur.com/RrhXmyC.jpg">
		
- 3. Kết quả

<img src="https://imgur.com/qt7RXkb.jpg">

<img src="https://imgur.com/l2vbbCS.jpg">
		
## ScaleUp Patterns
### Giới thiệu về auto scaling
- Định nghĩa: [Link](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html)
	
	Là dịch vụ của Amazon, cho phép theo dõi mở rộng, thu hẹp tài nguyên trên AWS (EC2, DynamoDB,  Aurora Replica) dựa trên các kịch bản được định sẵn phối hợp cùng các dịch vụ Amazon ELB, CloudWatch.
	Lưu ý: 
	
### Thực hành

Thực hành scale out yêu cầu tăng số lượng instance khi CPU vượt quá 75%

- 1. Tạo ELB tham số như hình

	<img src="https://imgur.com/au32Rjm.jpg">

	<img src="https://imgur.com/Rfj1KIh.jpg">

- 2. Tạo Lauch Configuration có `user data` được định nghĩa như sau:


			#! /bin/bash
			sudo apt-get upgrade
			sudo apt-get update -y
			sudo apt-get install awscli -y
			sudo apt-get install apache2 -y
			sudo apt-get install stress -y
			sudo service apache2 start
			sudo echo "public ip is $(curl http://169.254.169.254/latest/meta-data/public-ipv4), " >> hung.txt
			sudo echo "instance id is $(curl  http://169.254.169.254/latest/meta-data/instance-id)," >> hung.txt
			sudo echo "instance-type is $(curl  http://169.254.169.254/latest/meta-data/instance-type) " >> hung.txt
			sudo cat hung.txt > /var/www/html/index.html
	
Ý nghĩa: Khi truy cập web thông qua ELB để kiểm tra ta đang truy cập vào instance nào, `stress` để test tăng CPU

<img src="https://imgur.com/sd8PM3c.jpg">
	

- 3. Tạo AutoScaling Group tham số như dưới, **chú ý tích chọn Enable CloudWatch Detail Monitoring**
	- a. Activity / Gửi [notification]() qua text messae khi có sự kiện từ Autoscaling (sẽ bổ sung ở bài sau). Tại giao diện này cũng có thể theo dõi được sự kiện đã xảy ra 

		<img src="https://imgur.com/gxDTR5L.jpg">
	
	- b. Tham số `Increase` và tham số `Decrease` *đặt tên hơi lẫn :)*

		<img src="https://imgur.com/TrFp8Wl.jpg">

		Chú ý tại giao diện này ta có thể đặt lịch cho việc autoscaling (ví dụ này không có nhé :) )

		<img src="https://imgur.com/SZQuoD4.jpg">
	
	- c. Ngoài ra còn có giao diện `Instance Management` và `Monitoring` để quản lý và theo dõi Instance cũng như autoscaling event
		<img src="https://imgur.com/cl9treu.jpg">

- 4. Kiểm tra hoạt động của `ELB` tạo ở bước [1]() bằng cách truy cập browser đến DNS link của ELB or dùng lệnh như sau `curl --silent link`:
 	Kết quả với lệnh `curl` & web browser
	<img src="https://imgur.com/gTQYGHz.jpg">

	<img src="https://imgur.com/WHm0Nyw.jpg">
- 5. SSH vào instance và tăng CPU bằng lệnh `stress --cpu 6900 --timeout 240` --> `:( test mất 2 giờ để tìm đc con số chân lý :(`
	Mục đích tăng CPU lên ngưỡng cao để `Cloudwatch` detect và issue `alarm`.
	<img src="https://imgur.com/oj1yY0C.jpg">

- 6. Biểu đồ của `Cloudwatch` `EC2` và `AutoScale`

	<img src="https://imgur.com/BBfModT.jpggit c">
	
- 7. Lúc này 1 instance được khởi tạo thêm 

	<img src="https://imgur.com/QAs3PSI.jpg">
	Kiểm tra ELB thì 2 instance đều đã được tự động thêm vào :)

	<img src="https://imgur.com/KQKLxdE.jpg">
