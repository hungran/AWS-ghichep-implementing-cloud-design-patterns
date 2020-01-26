Vũ Mạnh Hùng
vmhung290791@gmail.com
- [Nguồn](https://www.amazon.com/Implementing-Cloud-Design-Patterns-AWS-ebook/dp/B00WX3W43I)
# GHI CHÉP QUÁ TRÌNH ĐỌC VÀ THỰC HÀNH CUỐN Implement_Cloud_Design_Patterns
## MỤC LỤC
- [Giới thiệu về Vagrant](https://github.com/hungran/AWS-ghichep-implementing-cloud-design-patterns#gi%E1%BB%9Bi-thi%E1%BB%87u-v%E1%BB%81-vagrant) / Công cụ được nhắc đến trong cuốn
- [Snapshoot patterns](https://github.com/hungran/AWS-ghichep-implementing-cloud-design-patterns#snapshoot-patterns)

*Môi trường yêu cầu*:

	- Tài khoản AWS (khuyên dùng tài khoản free)
	- Ubuntu 16.04 LTS freetier, t2.micro

### Giới thiệu về Vagrant
- Là công cụ xây dựng, quản lý máy ảo, có thể chạy trên ubuntu, macOS và Windows
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

- Tham khảo [Vagrantfile](https://github.com/hungran/AWS-ghichep-implementing-cloud-design-patterns/blob/master/Vagrantfile)s sample ở đây :)
	- Một số lệnh cơ bản của vagrant:
		1. `vagrant up --provider=aws` chạy vagrant với aws từ `vagrantfile`
		2. `vagrant reload --provision` khởi động lại VM
		3. `aws.user_data` dùng để định nghĩa bootstrap thay vì dùng [provision](https://www.vagrantup.com/intro/getting-started/provisioning.html)
		4. `vagrant destroy` terminate vm
### Snapshoot patterns
- Tạo snapshoot `ebs` đẩy vào s3 từ running instance
	- 1. Khởi chạy ec2 instance từ vagrantfile bằng lệnh `vagrant up --provider=aws`
	- 2. Vào giao diện GUI console -> tìm đến EC2 -> chọn instance đang chạy, tìm đến phần **description** -> trỏ chuột và chọn đến link **volume-id** có dạng **vol-xxx**
	- 3. Tại giao diện Volumes section, chuột phải chọn Volume cần snapshoot chọn **Create Snapshot**
- Trong quá trình tạo EC2 cho lab Snapshoot patterns bằng vagrantfile & bootstrap có nội dung như dưới
	`
	#! /bin/bash
sudo apt-get upgrade
sudo apt-get update -y
sudo apt install awscli -y
sudo apt-get install apache2 -y
sudo service apache2 start
sudo echo "public ip is $(curl http://169.254.169.254/latest/meta-data/public-ipv4), " >> hung.txt
sudo echo "instance id is $(curl  http://169.254.169.254/latest/meta-data/instance-id)," >> hung.txt
sudo echo "instance-type is $(curl  http://169.254.169.254/latest/meta-data/instance-type) " >> hung.txt
sudo cat hung.txt > /var/www/html/index.html
	`
- CMD kiểm tra instances bằng aws cli bằng lệnh `aws ec2 describe-instances`
	Kết quả như hình
	<img src ="https://imgur.com/TO2tulA.jpg">
- Tuy nhiên khi mở trên trình duyệt web lại ra kết quả public-ipv4 khác @@ --> Cần lắm 1 lời giải thích ở đây!!!
	<img src ="https://imgur.com/Gm0ZqDO.jpg">