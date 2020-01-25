Vagrant.configure("2") do |config|
  config.vm.box = "dummy"
  config.vm.provision :shell, path: "C:/HashiCorp/bootstrap.sh"
  config.vm.provider :aws do |aws, override|
    aws.access_key_id = "XXXXXXXXX"
    aws.secret_access_key = "XXXXXXX"
#    aws.session_token = "SESSION TOKEN"
#    aws.keypair_name = "KEYPAIR NAME"
	aws.region = "ap-southeast-1"
	#ubuntu 16.04 LTS
    aws.ami = "ami-0ee0b284267ea6cde"
	aws.instance_type = "t2.micro"
	aws.subnet_id = "subnet-04fa887456cd2ab68"
	aws.associate_public_ip = true
	aws.private_ip_address = "172.31.16.200"
	aws.security_groups = "sg-04625159d166466da"
	aws.tags = {
		'Name' => 'Hung_Test_Vagrantfile_Bootstrap_02'
	}
	aws.keypair_name = "hung20190616"
    override.ssh.username = "ubuntu"
    override.ssh.private_key_path = "C:/HashiCorp/hung20190616.pem"
  end
end