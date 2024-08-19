# WriteUp-Los-40-Ladrones
# [LOS 40 LADRONES](https://dockerlabs.es/#/)

# CONECTIVIDAD CON LA MAQUINA
Haremos un Ping para ver si tenemos respuesta activa de nuestro objetivo

# ESCANEO DE PUERTOS
Haremos un escaneo de puertos para ver cuales estan abiertos 
<br>
<br>
![image](https://github.com/user-attachments/assets/5f411fc5-62b3-49e6-803f-aca34f8808a2)
<br>
Vemos que tenemos el puerto 80 abierto por lo que vamos a ver su contenido en la web
<br>
Vemos que es una pagina de APACHE por lo que procederemos entonces a buscar sub-dominios

# BUSQUEDA DE SUB-DOMINIOS
En esta ocacion usare GOBUSTER para la busqueda de subd-ominios
<br>
`gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt -t 20 -x html,php,txt,php.bak`
<br>
<br>
![image](https://github.com/user-attachments/assets/9b5e2e1c-7f08-49c1-8412-dc57e500df21)
<br>
Vemos que encontramos un sub-dominio /qdefense.txt
<br>
Entramos a el y nos encontramos con lo siguiente
<br>
<br>
![image](https://github.com/user-attachments/assets/a3177208-77c6-4e1f-8494-6f617bdb8f46)
<br>
Obtenemos un usuario "toctoc" y una secuencia de numeros

# ESCANEO COMPLETO DE PUERTOS
Haremos un escaneo de todos los puertos de la maquina atacante para ver si tenemos algun puerto que este filtrado
<br>
`nmap --top-ports 25T -n -T4 172.17.0.2`
<br>
<br>
![image](https://github.com/user-attachments/assets/88e43c67-b6fd-48c3-b98a-be017c2b8cb0)
<br>
Vemos que hay muchos puertos filtrados, puede ser que algun puerto necesita un poco de ayuda para que sea abierto

# HERRAMIENTA KNOCK
Usaremos la herramienta knock para hacer un [PORT-KNOCKING](https://www.zonasystem.com/2022/08/port-knocking-ocultar-y-mejorar-la-seguridad-en-conexiones-ssh.html) con los numeros anteriormente obtenidos
<br>
`knock 172.17.0.2 7000 8000 9000 -v`
<br>
<br>
![image](https://github.com/user-attachments/assets/a3153af9-b7c8-41ff-91b6-a3f9657ee938)
<br>
<br>
Volveremos hacer un escaneo a todos los puertos de nuestra maquina objetivo y veremos que el puerto 22 (SSH) ahora esta abierto
<br>
<br>
![image](https://github.com/user-attachments/assets/94e957ed-6f45-4854-a2b5-d7102acffb14)


# FUERZA BRUTA CON HYDRA
Anteriormente obtuvimos un usuario "toctoc", veremos si lo podemos usar para entrar al servidor SSH
<br>
Usaremos la herramienta HYDRA para encontrar la contraseña del usuario
<br>
En este caso use el diccionario que tendremos en la carpeta `metasploit -> unix_password.txt` ya que `rockyou.txt` no me encontro la clave
<br>
`hydra -l toctoc -P /usr/share/wordlists/metasploit/unix_passwords.txt ssh://172.17.0.2 -t 10`
<br>
<br>
![image](https://github.com/user-attachments/assets/0a71b3b2-61cc-4cc3-a034-21ad55c5a5ae)


# CREDENCIALES DE USUARIO
Usuario: toctoc 
<br>
Contraseña: kittycat

# CONEXION SSH
Ya que tenemos el usuario y contraseña nos contectaremos al servidor SSH <br>
Iniciaremos una conexxion `ssh toctoc@172.17.0.2` <br>
Si nos sale el siguiente WARNING <br> <br>
![image](https://github.com/user-attachments/assets/49ba88fa-5a1d-45a6-bf13-115fc08e9504)
<br> <br>
Tendremos que colocar el comando que nos indica abajo `ssh-keygen -f '/root/.ssh/known_hosts' -R '172.17.0.2'`
<br> 
Una vez hecho esto volveremos a intentar conectarnos y ahora si nos pedira la contraseña <br>
Una vez dentro haremos un `whoami`para saber nuestro usuario y `sudo -l` para ver que comando podemos ejecutar sin privilegios <br> <br>
![image](https://github.com/user-attachments/assets/5d1c9265-c21b-4e7a-9f8f-626309fadfd0)

# ESCALADA DE PRIVILEGIOS
Como vimos anteriormente tenemos la opcion de poder usar el archivo bash que se encuentra en la carpeta /opt -> `/opt/bash` <br>
Por lo tanto con este comando abriremos una "nueva terminal bash" con los privilegios de usuario root <br>
Para esto usaremos el comando `sudo /opt/bash`
<br>
<br>
![image](https://github.com/user-attachments/assets/e66593a2-c5cb-46c3-a14d-fb3d375120fe)
<br>
<br>
Y luego un `whoami` para verificar si somos root
<br>
<br>
![image](https://github.com/user-attachments/assets/730c6860-1020-4ff1-8f05-e6eca6e57be2)


