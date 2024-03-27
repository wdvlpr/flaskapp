Apache2 + Flask + Ubuntu 20.04

Instalando pacotes necessários do Ubuntu

Vamos primeiro instalar os seguintes pacotes necessários:

$ sudo apt-get install python3
$ sudo apt-get install python3-pip
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi-py3

Instalando pacotes necessários para o aplicativo Flask:

$ sudo pip3 install flask, numpy, pandas

Configuração do aplicativo Flask com servidor Apache usando WSGI:

$ sudo mkdir /var/www/html/flaskapp

Nosso diretório flaskapp deve ter dois arquivos importantes a seguir:

flaskapp.py (arquivo raiz para aplicativo flask)

# flaskapp.py
# This is a "hello world" app sample for flask app. You may have a different file.
from flask import Flask
app = Flask(__name__)
@app.route('/') 
def hello_world():
   return 'Hello from Flask!' 
if __name__ == '__main__':
   app.run()
   

flaskapp.wsgi (arquivo raiz para wsgi, que iremos configurar com o servidor apache para executar flaskapp)

import sys 
sys.path.insert(0, '/var/www/html/flaskapp')
from flaskapp import app as application


Depois de criar o diretorio e os arquivos temos que adicionar as segintes configurações em /etc/apache2/sites-enabled/000-default.conf para servir nosso flaskapp:

$ sudo vi /etc/apache2/sites-enabled/000-default.conf

<VirtualHost *:8000>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        
		WSGIDaemonProcess flaskapp threads=5
        WSGIScriptAlias / /var/www/html/flaskapp/flaskapp.wsgi
        WSGIApplicationGroup %{GLOBAL}
        <Directory flaskapp>
             WSGIProcessGroup flaskapp
             WSGIApplicationGroup %{GLOBAL}
             Order deny,allow
             Allow from all 
        </Directory>		
		
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        Header set Access-Control-Allow-Origin "*"
</VirtualHost>

E em /etc/apache2/ports.conf adicionar as segintes configurações:

$ sudo vi /etc/apache2/ports.conf

# If you just change the port or add more ports here, you will likely also
# have to change the VirtualHost statement in
# /etc/apache2/sites-enabled/000-default.conf

Listen 80
Listen 8000

<IfModule ssl_module>
<------>Listen 443
</IfModule>

<IfModule mod_gnutls.c>
<------>Listen 443
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet


Reinicie o servidor Apache executando o seguinte comando:

$ sudo service apache2 restart

Agora acesse:

http://localhost:8000/
