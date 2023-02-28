Docker: Utilização prática no cenário de Microsserviços
Denilson Bonatti, Instrutor - Digital Innovation One

Muito se tem falado de containers e consequentemente do Docker no ambiente de desenvolvimento. Mas qual a real função de um container no cenários de microsserviços? Qual a real função e quais exemplos práticos podem ser aplicados no dia a dia? Essas são algumas das questões que serão abordadas de forma prática pelo Expert Instructor Denilson Bonatti nesta Live Coding. IMPORTANTE: Agora nossas Live Codings acontecerão no canal oficial da dio._ no YouTube. Então, já corre lá e ative o lembrete! Pré-requisitos: Conhecimentos básicos em Linux, Docker e AWS.


<p>Será utilizado três VMs Ubuntu Server 22.04 em KVM. Um para o Master, com duas placas de rede e outros dois Workes com uma placa de rede cada em NAT.</p>

<h3>Preparando o ambiente</h3>

<h4>MASTER, WORKER01, WORKER02</h4>

<p>apt install apt-transport-https ca-certificates curl software-properties-common -y</br>
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -</br>
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu jammy stable" -y</br>
apt install docker-ce -y</br>
systemctl status docker #OK!</br></p>


<h3>1° Cenário - 1 Container</h3>

<h4>MASTER</h4>

<p>docker volume create app</br>
docker volume create data</br>
docker run -e MYSQL_ROOT_PASSWORD=Senha123 -e MYSQL_DATABASE=meubanco --name mysql-A -d -p 3306:3306 --mount type=volume,src=data,dst=/var/lib/mysql/ mysql:5.7</br>
"Criar a table no banco"</br>
"Criar index.php em app" - Não esquecer de atualizer o IP do web-server</br>
docker run --hostname web-server-master --name web-server -dt -p 80:80 --mount type=volume,src=app,dst=/app/ webdevops/php-apache:alpine-php7</br>
"Estressar o container pelo site: https://loader.io/"</br></p>


<h3>2° Cenário - Cluster Swarm (10 Containers)</h3>

<h4>MASTER</h4>

<p>docker rm -f web-server</br>
docker swarm init --advertise-addr 192.168.x.x #Rede NAT do KVM</br>
docker node ls #Verificar os nos</br>
docker service create --name web-server --replicas 10 -dt -p 80:80 --mount type=volume,src=app,dst=/app/ webdevops/php-apache:alpine-php7</br>
docker service ps web-server</br>
apt install nfs-server -y</br>
/var/lib/docker/volumes/app/_data 192.168.x.0/24(rw,sync,subtree_check) >> /etc/exports</br>
exportfs -ar</br>
showmount -e</br>
mkdir /proxy</br>
cd /proxy/</br>
"Criar nginx.conf para servir de proxy"</br>
"Criar dockerfile para criar uma imagem do nginx com o nginx.conf"</br>
docker build -t proxy-app .</br>
docker run --name my-proxy-app --hostname my-proxy-app -dti -p 4500:4500 proxy-app</br></p>

<h4>WORKER01, WORKER02</h4>

<p>docker swarm join --token SWMTKN-1-9792ac8fc0be5d8f9784a22f939c8b16233a8a2377077e4e946cd833efca6414 192.168.x.x:2377</br>
apt install nfs-common -y</br>
mount -o v3 192.168.x.x:/var/lib/docker/volumes/app/_data /var/lib/docker/volumes/app/_data</br></p>

<h4>Teste Final</h4>
<p>"Estressar o cluster pelo site: https://loader.io/"</p>
