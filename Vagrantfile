Vagrant.configure("2") do |config|
  config.vm.box = "perk/ubuntu-2204-arm64"
  config.vm.box_version = "20250702"

  # =========================================================================
  # MÁY 1: MASTER NODE (Cổng SSH: 52222)
  # =========================================================================
  config.vm.define "cka-master" do |master|
    master.vm.hostname = "cka-master"
    master.vm.network "forwarded_port", guest: 22, host: 52222, id: "ssh", auto_correct: true
    master.vm.network "forwarded_port", guest: 6443, host: 6443, auto_correct: true

    master.vm.provider "qemu" do |qe|
      qe.arch = "aarch64"
      qe.cpus = 2              
      qe.memory = 3072         
      qe.ssh_port = 52222      # ÉP MASTER SỬ DỤNG CỔNG 52222
    end
  end

  # =========================================================================
  # MÁY 2: WORKER NODE 1 (Cổng SSH: 52223)
  # =========================================================================
  config.vm.define "cka-worker1" do |worker1|
    worker1.vm.hostname = "cka-worker1"
    worker1.vm.network "forwarded_port", guest: 22, host: 52223, id: "ssh", auto_correct: true

    worker1.vm.provider "qemu" do |qe|
      qe.arch = "aarch64"
      qe.cpus = 1
      qe.memory = 1536
      qe.ssh_port = 52223      # ÉP WORKER 1 SỬ DỤNG CỔNG 52223 (Bỏ cổng 50022)
    end
  end

  # =========================================================================
  # MÁY 3: WORKER NODE 2 (Cổng SSH: 52224)
  # =========================================================================
  config.vm.define "cka-worker2" do |worker2|
    worker2.vm.hostname = "cka-worker2"
    worker2.vm.network "forwarded_port", guest: 22, host: 52224, id: "ssh", auto_correct: true

    worker2.vm.provider "qemu" do |qe|
      qe.arch = "aarch64"
      qe.cpus = 1
      qe.memory = 1536
      qe.ssh_port = 52224      # ÉP WORKER 2 SỬ DỤNG CỔNG 52224 (Bỏ cổng 50022)
    end
  end
end
