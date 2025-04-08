# Simulación de Ataque y Defensa con DevSecOps

## Objetivo  
Demostrar cómo detectar y mitigar vulnerabilidades en una aplicación web (DVWA) usando Docker, Trivy y GitHub Actions.

## Herramientas  
- **Docker**: Contenedor con DVWA (aplicación vulnerable en la cual vamos a realizar nuestras pruebas).  
- **Trivy**: Escáner de vulnerabilidades. (se puede ejecutar de forma local o de forma mas automatizada en github actions despues de cada push)  
- **GitHub Actions**: Automatización del escaneo.  

***** Pasos para replicar la PoC *****  
1. Desplegar DVWA en Docker desde powershell:
   docker run -d -p 8080:80 --name dvwa vulnerables/web-dvwa
   
2. En su navegador ingresar a "localhost:8080"
   
3. Loguearse en la pag con las siguientes credenciales:
   Username= admin     Password= password
   
4. Una vez dentro de DVWA seleccionamos la opcion de "Create/Reset Database" (se encuentra al final del Setup Check)
   y esperamos unos segundos a que nos redirija al login de la pagina

5. Loguearse en la pag con las mismas credenciales:
   Username:admin     Password:password

6. Una vez que ingresamos nuevamente a DVWA, vamos a ver un nuevo menu al lado izquierdo de nuestra pantalla,
   vamos a seleccionar la opcion que dice "SQL Injection"

7. En la barra de busqueda de "User ID" vamos a ingresar el siguiente comando: 1' OR '1'='1
   (esto confunde a la pag y va a hacer que nos muestre una lista de usuarios filtrados. Hacking EXITOSO!)

8. Ahora podemos ejecutar un scaneo de TRIVY:
   docker run --rm aquasec/trivy image --severity CRITICAL vulnerables/web-dvwa

9. Creamos un repositorio en github con el nombre Proyecto-DevSecOps, y lo clonamos de forma local en cualquier carpeta de nuestro agrado
   git clone https://github.com/*cambiar este espacio por su nombre de usuario*/Proyecto-DevSecOps.git
   cd Proyecto-DevSecOps

10. Dentro de este repositorio que acabamos de clonar, que calramente va a estar vacio, creamos lo siguientes archivos (open git bash here):
    mkdir -p .github/workflows
    touch .github/workflows/scan.yml

11. Una vez creados nos dirigimos a github>workflows>scan, y nos va a abirir un editor de texto, en mi caso fue Visual Studio Code, y pegamos el siguiente codigo:
    name: Docker Image Security Scan
    on: [push]  # Se ejecuta al hacer git push

    jobs:
      scan:
       runs-on: ubuntu-latest
       steps:
        - name: Checkout code
          uses: actions/checkout@v2

        - name: Install Trivy
          run: |
            sudo apt-get update
            sudo apt-get install -y wget apt-transport-https gnupg lsb-release
    wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
          echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
          sudo apt-get update
          sudo apt-get install -y trivy

      - name: Scan Docker Image
        run: |
          trivy image --severity CRITICAL vulnerables/web-dvwa

13. Guardamos los cambios, y subimos los cambios a github:
    git add .github/workflows/scan.yml
    git commit -m "Add GitHub Actions escaneo de seguridad"
    git push origin main

14. Nos dirigimos a GitHub>ingresamos en nuestro depositorio>ingresamos en Actions>workflow en ejecicion o completado>Scan>Scan Docker Image
    y ya vamos a poder ver el reporte del escaneo de TRIVY  

