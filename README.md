# ELK Stack with Docker Documentation

Repository ini berisi tentang tutorial instalasi dan konfigurasi ELK Stack

# Spesification

**OS** : Ubuntu Server 20.04


# Docker Installation

Dalam dokumentasi resmi yang di publish disini [[1]](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html). Elasticsearch menyediakan berbagai macam pilihan metode/packages untuk berbagai environment seperti `Windows .zip archive`,`deb`,`rpm`,`docker`, dan `Elastic Cloud on Kubernetes`. Dan untuk dokumentasi ini kita akan menggunakan packages `deb` untuk instalasi elasticsearch.

Download dan install public signing key:

```bash
sudo apt-get update
```
```bash
 sudo apt-get install ca-certificates gnupg lsb-release
```

Install terlebih dahulu apt-transport-https package on Debian sebelum lanjut:

```bash
sudo mkdir -p /etc/apt/keyrings
```

Simpan repository yang di definisikan di `/etc/apt/sources.list.d/elastic-8.x.list`:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Lakukan update repository dan install elasticsearch

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
sudo apt-get update
```

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
```bash
sudo docker run hello-world
```



# Konfigurasi Clone Repository Docker-ELK

```bash
git clone
```





# Instalasi & Konfigurasi Kibana

Untuk kibana ini akan kita install pada `Node-1`

Instal kibana dengan menggunakan perintah dibawah ini

```bash
apt-get install kibana -y
```

buka file konfigurasi kibana dan lakukan konfigurasi sesuai dengan kebutuhan

```bash
nano /etc/kibana/kibana.yml
```

Uncomment syntax dibawah ini untuk mendefinisikan port dari kibana

```bash
#server.port: 5601
```

![Cluster Screenshot](./gambar/kibana-port.JPG)

uncomment syntax dibawah ini untuk mendefinisikan network host sesuai dengan IP Server yang dipakai

```bash
#server.host: "localhost"
```

![Cluster Screenshot](./gambar/kibana-host.JPG)

uncoment syntax dibawah ini untuk mengkoneksikan kibana dengan elasticsearch

```bash
#elasticsearch.hosts: ["http://localhost:9200"]
```

![Cluster Screenshot](./gambar/kibana-elastic-host.JPG)

setelah di konfigurasi sesuai dengan kebutuhan, lakukan save dan keluar dari text editor

jalankan service kibana dengan perintah dibawah ini

```bash
systemctl start kibana
systemctl enable kibana
```

akses kibana di web browser dengan alamat `http://<ip host>:5601`

```bash
http://10.10.10.101:5601
```

![Kibana Screenshot](./gambar/kibana.JPG)

## Instalasi & Konfigurasi Logstash

Untuk Logstash ini akan kita install di VM yang berbeda dengan cluster elasticsearch. Maka dari ini kita perlu menambahkan repository terlebih dahulu

Download dan install public signing key:

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```

Install terlebih dahulu apt-transport-https package on Debian sebelum lanjut:

```bash
sudo apt-get install apt-transport-https
```

Simpan repository yang di definisikan di `/etc/apt/sources.list.d/elastic-8.x.list`:

```bash
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

Lakukan update repository dan install logstash dengan perintah dibawah ini

```bash
sudo apt-get update && sudo apt-get install logstash -y
```

konfigurasi logstash untuk inputan dari filebeat

```bash
nano /etc/logstash/conf.d/01-beats-input.conf
```

masukkan syntax dibawah ini untuk mengkonfigurasi inputan dari filebeat

```bash
input {
  beats {
    port => 5044
  }
}
```

konfigurasi logstash untuk outputan ke elasticsearch

```bash
nano /etc/logstash/conf.d/02-elasticsearch-output.conf
```

masukkan syntax dibawah ini untuk mengkonfigurasi outputan ke elasticsearch

```bash
output {
  if [@metadata][pipeline] {
    elasticsearch {
      hosts => ["10.10.10.101:9201"]
      manage_template => false
      index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
      pipeline => "%{[@metadata][pipeline]}"
    }
  } else {
    elasticsearch {
      hosts => ["10.10.10.101:9201"]
      manage_template => false
      index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    }
  }
}
```

save dan keluar dari text editor. verifikasi konfigurasi logstash dengan perintah dibawah ini

```bash
sudo -u logstash /usr/share/logstash/bin/logstash --path.settings /etc/logstash -t
```

jika konfigurasi logstash berhasil, maka akan muncul output berikut
![verify logstash](./gambar/verify-logstash.JPG)

jalankan service logstash dengan perintah dibawah ini

```bash
systemctl start logstash
systemctl enable logstash
```

## Authors

- [Ahmad Naoval Annasik](https://www.github.com/octokatherine)
