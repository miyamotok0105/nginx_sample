
ユーザー



```
sudo apt-get install nginx gunicorn3
今回はここを使用
git clone https://github.com/wesbarnett/flask-project
cd flask-project
#単体で動くこと
gunicorn3 --chdir application -b :8080 app:app
```



### systemdの設定


```
sudo touch /etc/systemd/system/gunicorn.service
```


```:/etc/systemd/system/gunicorn.service
[Unit]
Description=gunicorn to serve flask-project
After=network.target

[Service]
WorkingDirectory=/home/ubuntu/flask-project
ExecStart=/usr/bin/gunicorn3 -b 0.0.0.0:8080 --chdir application app:app

[Install]
WantedBy=multi-user.target

```


```
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```


### Nginx設定


```
sudo rm /etc/nginx/sites-enabled/default
```


```:/etc/nginx/sites-available/flask-project.conf

server {
    listen 80;
    listen [::]:80;

    location / {
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host              $http_host;
        proxy_pass http://localhost:8080;
    }
}
```

```
sudo ln -s /etc/nginx/sites-available/flask-project.conf /etc/nginx/sites-enabled/

sudo systemctl restart nginx.service

#もし80を使ってたら止める
sudo lsof -i:80
sudo service apache2 stop

sudo systemctl restart nginx.service
sudo systemctl enable nginx

ローカルからも、外部からも接続できるはず
curl http://localhost:8080

```




## 参考

Run a Flask app on AWS EC2    

https://barnett.science/linux/aws/ansible/2020/05/28/ansible-flask.html

Deploying Python Web Applications with nginx and uWSGI Emperor    

https://chriswarrick.com/blog/2016/02/10/deploying-python-web-apps-with-nginx-and-uwsgi-emperor/

