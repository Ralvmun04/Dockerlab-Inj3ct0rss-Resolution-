<img width="947" height="496" alt="Captura desde 2026-05-28 23-55-38" src="https://github.com/user-attachments/assets/5df00825-11c7-434b-97fc-11244ece81eb" />


Comenzamos el laboratorio iniciando el laboratorio de **Dockerlabs** con el siguiente comando:
```
sudo bash auto_deploy.sh inj3ct0rss.tar
```

<img width="951" height="437" alt="Captura desde 2026-05-28 22-37-21" src="https://github.com/user-attachments/assets/9eb50448-0497-416b-806d-35f4cc5ea115" />

Una vez se despliegue el laboratorio hacemos un escaneo de puestos con la herramienta nmap, con las siguientes opciones:
```
sudo nmap 172.17.0.2 -p- --open -sS -sC -sV --min-rate 5000 -vvv -Pn -n
```
**-p-** : Escanea los 65535 puertos existentes.
**--open** : Muestra los puertos abiertos.
**-sS** : Realiza un escaneo semi-abierto (TCP SYN Scan).
**-sC** : Usa scripts para encontrar informacion.
**-sV** : Que nos informe de la version del servicio que se ejecute.
**--min-rate 5000** : Que envie 5000 paquetes por segundo (Escaneo rapido pero ruidoso).
**-vvv** : Triple verbose para ver la informacion en tiempo real.
**-Pn** : Omite el descubrimiento de hosts.
**-n** : Que desactive la resolucion DNS inversa.

<img width="951" height="437" alt="Captura desde 2026-05-28 22-39-01" src="https://github.com/user-attachments/assets/c6778302-3d99-4903-a183-aab2a9eaf36a" />

Como podemos ver tenemos abierto el **puerto 80** (correspondiente a http) y el **puerto 22** (correspondiente a ssh).

Al buscar la pagina web nos encontramos con una bienvenida y un CTF basado en la vulnerabilidad **SQLInjection**, pasamos la pagina por gobuster para buscar directorios ocultos pero no encontramos nada interesante:

<img width="955" height="708" alt="Captura desde 2026-05-28 22-42-17" src="https://github.com/user-attachments/assets/ee4fca63-58d1-4a45-8fa8-32e4b948f871" />

Le damos al boton de inicio de sesion y, antes que nada probamos y nos tiramos un triple:

<img width="955" height="708" alt="Captura desde 2026-05-28 22-43-03" src="https://github.com/user-attachments/assets/53761249-a7be-4cce-8bbb-94b452448446" />

y...

<img width="1257" height="238" alt="Captura desde 2026-05-28 22-43-30" src="https://github.com/user-attachments/assets/5d8c0372-371b-4689-8565-02430a5dedd7" />

**Bingo!!!**
Logramos entrar como el usuario **admin** gracias a la vulnerabilidad **SQLInjection**!!
al descubrir que este login es **vulnerable** al SQLInjection lo pasamos por **sqlmap** con las siguientes opciones:
```
sqlmap -u http://172.17.0.2/login.php --forms --dbs --batch --dump
```
**-u** : Indicamos la URL a atacar.
**--forms** : Le indicamos que esta frente a un formulario html.
**--dbs** : Enumere las bases de datos disponibles.
**--batch** : Ejecuta en modo automatico.
**--dump** : Escribe en pantalla los datos de las bases de datos que hayas encontrado.

Vemos que la herramienta tarda en sacar los datos, ya que va de letra en letra, esto es porque esta haciendo un ataque de Inyeccion SQL basado en **tiempo**, en el que le pide a la base de datos, de letra en letra, si esa es la suguiente letra correcta, si lo es, que espere, y si no, que lo rechace, esperamos un rato y **tenemos los resultados**:

<img width="494" height="247" alt="Captura desde 2026-05-28 22-44-48" src="https://github.com/user-attachments/assets/7e89ae77-7b27-4b88-a555-9aff86b7cf05" />

Vemos que hay un usuario con una password que nos da una pista, buscando el directorio encontramos lo siguiente:

<img width="879" height="310" alt="Captura desde 2026-05-28 22-47-51" src="https://github.com/user-attachments/assets/6e0302e8-88f9-4fbe-a9b3-d2d1bce5deae" />

Un **secret.zip** que, al descargar, vemos que nos pide una contrasena, probamos varias que vemos en la base de datos pero nos falla, por lo que lo pasamos por la herramienta **fcrackzip**:

<img width="946" height="115" alt="Captura desde 2026-05-28 22-48-52" src="https://github.com/user-attachments/assets/fa68b51c-914b-48be-bf41-6e8bb431c946" />

**Nos encuentra la contrasena**, dentro del .zip encontramos un archivo llamado **confidencial.txt**:

>You have to change your password ralf, I have told you many times, log into your account and I will change your password.
>Your new credentials are:
>ralf:supersecurepassword

Usamos estas credenciales en el puerto ssh y **tenemos acceso**, al buscar una forma de escalar privilegios, encontramos una opcion con **sudo -l**:

<img width="943" height="398" alt="Captura desde 2026-05-28 22-56-05" src="https://github.com/user-attachments/assets/fb72ab1e-da38-468b-b7a6-74767e0382eb" />

Buscando en **GTFObins** encontramos que para el binario **busybox** mediante sudo era necesario el siguiente comando:
```
sudo busybox ash
```
Adaptandolo al usuario con permisos de ejecucion de busybox finalmente usamos:
```
sudo -u capa /usr/local/bin/busybox /nothing/ash
```
Y Conseguimos acceso al usuario **capa**, donde buscando en sus archivos encontramos su usuario y contrasena, las cuales usamos para tener una terminal mas interactiva:

<img width="943" height="398" alt="Captura desde 2026-05-28 22-58-28" src="https://github.com/user-attachments/assets/556d7b4f-6d9e-4313-bdd5-4c93c9fd37a6" />

Al hacer **sudo -l** en busqueda de una escalada de privilegios en el usuario capa, nos encontramos con que podemos usar **cat** sin restricciones, por lo tanto se nos ocurre hacerlo en el **id_rsa del usuario root**:

<img width="943" height="398" alt="Captura desde 2026-05-28 22-58-28" src="https://github.com/user-attachments/assets/f1b1a46b-4d81-413f-b0ef-bcb8fc4aceb2" />

Nos funciona por lo tanto nos guardamos el **id_rsa** en un archivo llamado de la misma manera, al cual le daremos **permisos de administrador** y lo usaremos para el login de root de esta manera:

<img width="947" height="496" alt="Captura desde 2026-05-28 23-03-21" src="https://github.com/user-attachments/assets/f993eac0-6857-4d19-b94c-e825048c3aee" />

Finalmente en root encontramos la ultima **flag** y damos el laboratorio por **finalizado**.

En este laboratorio hemos aprendido lo peligroso que es una mala configuracion de algo tan simple como un formulario de Login, en el que el input del usuario puede llegar a conseguir el acceso a nuestras bases de datos, dejando al descubierto las partes mas vulnerables de nuestro sistema.
