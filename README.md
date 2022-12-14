# h6-kulkurin-projekti

Karvinen 2017: Vagrant Revisited – Install & Boot New Virtual Machine in 31 seconds (Suosittelen käyttämään tässä koneena 'vagrant init debian/bullseye64')
yksinkertainen ohje miten asentaa vagrantvirtualbox linuxilla ja kuinka ottaa se käyttöön 
$` sudo apt-get update`
$ `sudo apt-get install vagrant virtualbox`


Karvinen 2021: Two Machine Virtual Network With Debian 11 Bullseye and Vagrant
Ohjeet kuinka asentaa vagrant virtualbox linuxille ja windowsille.
$ `mkdir twohost/; cd twohost/`
$` nano Vagrantfile` jonne copypastetaan koodi teron sivuilta.
`vagrant init debian/bullseye64`
`vagrant up`

Karvinen 2018: Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux
Ohjeet salt masterin ja salt minion asennukseen ja käyttöön ottoon. Olemme jo aikaisemmin tällä kurssilla tutustuttu niihin.


###a) Vagrantvirtualbox via windows host

Ensiksi latasain vagrantin amd64 version koneelle ja asensin sen, jonka jälkeen otetaan käyttöön se windwosin PowerShellissä.
Luodaan uusi kansio projektille komennolla $` mkdir twohost/; cd twohost/`
Tämän jälkeen luodaan vagrantfile tekstitiedosto
`notepad.exe vagrantfile`
notepad ikkunaan copypastetaan 

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
# Copyright 2019-2021 Tero Karvinen http://TeroKarvinen.com

$tscript = <<TSCRIPT
set -o verbose
apt-get update
apt-get -y install tree
echo "Done - set up test environment - https://terokarvinen.com/search/?q=vagrant"
TSCRIPT

Vagrant.configure("2") do |config|
	config.vm.synced_folder ".", "/vagrant", disabled: true
	config.vm.synced_folder "shared/", "/home/vagrant/shared", create: true
	config.vm.provision "shell", inline: $tscript
	config.vm.box = "debian/bullseye64"

	config.vm.define "t001" do |t001|
		t001.vm.hostname = "t001"
		t001.vm.network "private_network", ip: "192.168.88.101"
	end

	config.vm.define "t002", primary: true do |t002|
		t002.vm.hostname = "t002"
		t002.vm.network "private_network", ip: "192.168.88.102"
	end
	
end
```

Tämän jälkeen ajetaan komento: `vagrant init debian/bullseye64`
jonka jälkeen ajetaan `vagrant up` komento.
Nyt vagrant on windowsillasi ja voit kirjautua virtuaalikoneellesi 
`vagrant ssh t001`
jos haluat poistua tilasta niin käytä komentoa exit.
Jos haluat poistaa kokonaan virtuaalikoneen, niin se onnistuu komennolla vagrant destroy

### b) Private network

Ensiksi luodaan uusi kansio c:/users/jarkko/twohost

Sitten loin uuden vagrant tiedoston `vagrant init` jonne lisäsin
```
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"

  config.vm.define "isanta" do |isanta|
		isanta.vm.hostname = "isanta"
		isanta.vm.network "private_network", ip: "10.0.2.15"
	end

	config.vm.define "renki1", primary: true do |renki1|
		renki1.vm.hostname = "renki1"
		renki1.vm.network "private_network", ip: "10.0.2.15"
	end
end
```
Sen jälkeen ajetaan `vagrant up`

kokeillaan muodostaa ssh yhteys renki1 kanssa ja pingataan isanta koneen ip osoite saadaan vastaus isannalta.

### C) Salt master slave over the network with the previous created assigment

Asennetaan salt-minion ja salt-master
 ```
   sudo apt-get update
   sudo apt-get install salt-minion
```
 ```
   sudo apt-get update
   sudo apt-get install salt-master
```

Muokataan sudoedit `/etc/salt/minion`
master: 10.0.2.15
id: renki1

Tämän jälkeen restarttasin minionin
`sudo systemctl restat salt-minion`
Sitten hyväksytään avain
`sudo salt-key -A`
