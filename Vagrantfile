# Описание параметров ВМ
MACHINES = {
  # Имя DV "pam"
  :"pam" => {
              # VM box
              :box_name => "generic/centos8s",
              #box_version
              #:box_version => "20210210.0",
              # Количество ядер CPU
              :cpus => 2,
              # Указываем количество ОЗУ (В Мегабайтах)
              :memory => 1024,
              # Указываем IP-адрес для ВМ
              :ip => "192.168.57.11",
            }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    # Отключаем сетевую папку
    config.vm.synced_folder ".", "/vagrant", disabled: true
    # Добавляем сетевой интерфейс
    config.vm.network "private_network", ip: boxconfig[:ip]
    # Применяем параметры, указанные выше
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.box_version = boxconfig[:box_version]
      box.vm.host_name = boxname.to_s

      box.vm.provider "virtualbox" do |v|
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
      end
      box.vm.provision "shell", inline: <<-SHELL
          #Разрешаем подключение пользователей по SSH с использованием пароля
          sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config
          #Перезапуск службы SSHD
          systemctl restart sshd.service
	  sudo useradd otusadm && sudo useradd otus
          sudo echo "otus" | sudo passwd --stdin otusadm && echo "otus" | sudo passwd --stdin otus
          sudo groupadd -f admin
          sudo usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin
          sudo wget -P /usr/local/bin/ https://github.com/sashant157/homework15/raw/main/login.sh
          sudo chmod +x /usr/local/bin/login.sh
          sudo sed -i '/auth       include      postlogin/a auth       required     pam_exec.so /usr/local/bin/login.sh' /etc/pam.d/sshd
  	  SHELL
    end
  end
end

