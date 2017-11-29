# adapted from: https://github.com/poldrack/fmri-handbook-2e-code/blob/master/Vagrantfile
VAGRANTFILE_API_VERSION = "2"

$script = <<SCRIPT


# update the repositories
sudo apt-get update

# install basic packages (R, FSL, AFNI, ANTS)
sudo apt-get install -y \
r-base \
fsl-core \
fsl-atlases \
afni \
ants

# configure fsl
echo "# FSL config" >> ~/.bashrc
echo "source /usr/share/fsl/5.0/etc/fslconf/fsl.sh" >> ~/.bashrc

# configure afni
echo "# AFNI config" >> ~/.bashrc
echo "source /etc/afni/afni.sh" >> ~/.bashrc


# install directory
install_dir=${HOME}/.packages
mkdir -p ${install_dir}

# install freesurfer
wget ftp://surfer.nmr.mgh.harvard.edu/pub/dist/freesurfer/6.0.0/freesurfer-Linux-centos6_x86_64-stable-pub-v6.0.0.tar.gz -O ${install_dir}/freesurfer-Linux-centos6_x86_64-stable-pub-v6.0.0.tar.gz
tar xvf ${install_dir}/freesurfer-Linux-centos6_x86_64-stable-pub-v6.0.0.tar.gz -C ${install_dir}

# get freesurfer license (James Kent provided)
wget https://www.dropbox.com/s/rqxnkphzkp516o5/freesurfer_license.txt?dl=0 -O ~/.packages/freesurfer/freesurfer_license.txt

# configure freesurfer
# https://surfer.nmr.mgh.harvard.edu/fswiki/DownloadAndInstall
echo "# FREESURFER config" >> ~/.bashrc
echo "export FREESURFER_HOME=${HOME}/.packages/freesurfer" >> ~/.bashrc
echo 'source $FREESURFER_HOME/SetUpFreeSurfer.sh' >> ~/.bashrc

# remove freesurfer tar file
rm ${install_dir}/freesurfer-Linux-centos6_x86_64-stable-pub-v6.0.0.tar.gz


# install anaconda
# download installer
wget http://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ${install_dir}/miniconda.sh
# make the installer executable
chmod +x ${install_dir}/miniconda.sh
# run the miniconda installer
${install_dir}/miniconda.sh -p ${install_dir}/miniconda3 -b
# remove the installer script (don't need it anymore)
rm ${install_dir}/miniconda.sh
# update our environment to use miniconda programs
echo "export PATH=${install_dir}/miniconda3/bin:\$PATH" >> ~/.bashrc

# install nipype and nipy
${install_dir}/miniconda3/bin/conda install --yes -c conda-forge \
nipype \
nipy

# install packages not in conda-forge
${install_dir}/miniconda3/bin/conda install --yes \
jupyter \
seaborn \
scikit-learn \
rpy2

# install packages via pip
${install_dir}/miniconda3/bin/pip install \
niwidgets \
jupyter_contrib_nbextensions

# enable widgets using jupyter
${install_dir}/miniconda3/bin/jupyter nbextension enable --py widgetsnbextension
${install_dir}/miniconda3/bin/jupyter nbextension enable --py --sys-prefix ipyvolume

# download atom text editor
wget https://atom.io/download/deb -O ${install_dir}/atom-amd64.deb

# install atom text editor, will have unmet dependencies, solved by apt-get
sudo dpkg -i /home/brain/.packages/atom-amd64.deb
sudo apt-get install -y -f

# remove deb file (don't need it anymore)
rm ${install_dir}/atom-amd64.deb

# install docker
# https://docs.docker.com/engine/installation/linux/docker-ce/debian/#install-docker-ce-1

# download the prereqs
sudo apt-get install -y \
apt-transport-https \
ca-certificates \
curl \
gnupg2 \
software-properties-common

# get Docker's official GPG key
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo apt-key add -

# verify the fingerprint
sudo apt-key fingerprint 0EBFCD88

# add the repository
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable"

# update the package manager to include Docker
sudo apt-get update

# install docker
sudo apt-get install -y docker-ce

# remove any packages that are orphaned (e.g. unused by other packages)
sudo apt-get clean -y
sudo apt-get autoclean -y
sudo apt-get autoremove -y

# not sure what this does yet
sudo VBoxClient --display -d
sudo VBoxClient --clipboard -d

SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
   config.ssh.forward_x11 = true
   # base box created from neurodebian
   config.vm.box = "jdkent/NeuroDebian64"

   # my second release
   config.vm.box_version = "0.0.2"

   # changes default username of vagrant
   config.ssh.username = "brain"
   config.ssh.password = "neurodebian"
   config.ssh.insert_key = true
   # configure virtualbox
   config.vm.provider :virtualbox do |vb|
       # http://www.virtualbox.org/manual/ch03.html#settings-motherboard
       # required for 64-bit guest operating systems
       vb.customize ["modifyvm", :id, "--ioapic", "on"]
       # changes memory to (a little less than) 3GB
       vb.customize ["modifyvm", :id, "--memory", "3000"]
       # grants 2 cpus to user
       vb.customize ["modifyvm", :id, "--cpus", "2"]
       # https://www.virtualbox.org/manual/ch09.html#idm8260
       # allows you to resize the vm and change resolution accordingly?
       vb.customize ["setextradata", :id, "GUI/MaxGuestResolution", "any"]
       # https://www.virtualbox.org/manual/ch09.html#nat-adv-dns
       vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
       # https://www.virtualbox.org/manual/ch09.html#nat-adv-dns
       vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
       # http://datasift.github.io/storyplayer/v2/tips/vagrant/speed-up-virtualbox.html
       vb.customize ["modifyvm", :id, "--nictype1", "virtio"]
       vb.customize ["modifyvm", :id, "--vram", "64"]
       # use a gui when booting up the vm (if false, it presumes you use ssh to
       # interact with the vm)
       vb.gui = true
       # Enable, if Guest Additions are installed, whether hardware 3D acceleration should be available
       # For using jupyter notebooks
       vb.customize ["modifyvm", :id, "--accelerate3d", "on"]
       # what the virtual machine will be called
       vb.name = "uiowa-mri-course-2018"

       if Vagrant.has_plugin?("vagrant-cachier")
           # Configure cached packages to be shared between instances of the same base box.
           # More info on the "Usage" link above
           config.cache.scope = :box
       end
   end
   # uncomment following line to allow syncing to local machine
   config.vm.synced_folder ".", "/vagrant"
   config.vm.provision "shell", :privileged => false, inline: $script
end
