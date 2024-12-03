ARL(Asset Reconnaissance Lighthouse)资产侦察灯塔系统
git clone https://github.com/TophantTechnology/ARL
cd ARL/docker/
docker-compose up -d 

启动docker
sudo service docker start
停止docker
sudo service docker stop
重启docker
sudo service docker restart

docker volume create --name=arl_db