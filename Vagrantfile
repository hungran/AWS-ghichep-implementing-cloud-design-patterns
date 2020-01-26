Vagrant.configure("2") do |config|
  config.vm.box = "dummy"
  # Using rsync by cygwin64 instead SMB share
  config.vm.allowed_synced_folder_types = [:rsync]	
  config.vm.synced_folder "./share", "/vagrant"
  config.vm.provider :aws do |aws, override|
    aws.access_key_id = File.read("access_key.txt")
    aws.secret_access_key = File.read("secret_access_key.txt")
#	aws.session_token = "SESSION TOKEN"
#	aws.keypair_name = "KEYPAIR NAME"
	aws.region = "ap-southeast-1"
	#ubuntu 16.04 LTS
    aws.ami = "ami-0ee0b284267ea6cde"
	aws.instance_type = "t2.micro"
	aws.availability_zone = "ap-southeast-1b"
	aws.subnet_id = "subnet-04fa887456cd2ab68"
	aws.associate_public_ip = true
	aws.private_ip_address = "172.31.16.200"
	aws.security_groups = "sg-04625159d166466da"
	aws.tags = {
		'Name' => 'Hung_Test_Vagrantfile_Bootstrap_02'
	}
	aws.keypair_name = "hung20190616"
	aws.user_data = File.read("bootstrap.txt")
#	Câu lệnh mặc định sử dụng user name
    override.ssh.username = "ubuntu"
#	Câu lệnh mặc định sử dụng keypem dùng khi ssh vào instance 
    override.ssh.private_key_path = "C:/HashiCorp/hung20190616.pem"
	config.ssh.username = 'ubuntu'
  end
end