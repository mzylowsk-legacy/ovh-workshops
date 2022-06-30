# RoadShows 2022

#### 1. Przechadzka po managerze
- Wprowadzenie
- Metody płatności
- Utworzenie projektu public cloud

#### 2. Pierwsza instancja
- Wygenerowanie klucza SSH
```
ssh-keygen -t ecdsa -f demo
```
- Uruchomienie pierwszej instancji (D2-2, publiczne IP)
- Instalacja dokera za pomocą cloud-init'a:
```
#include https://get.docker.com/
```
- Logowanie się do uruchomionej maszyny wirtualnej
- ip a
```
ip a
```
- Uruchmienie kontenera 'httpd' z naszym własnym tekstem na stronie:
```
sudo docker run -d -p 80:80 --name www-server httpd && sudo docker exec www-server sh -c "echo 'Hello from OVH workshops :)' > htdocs/index.html"
```
- Potwierdzenie, że działa łączność po protokole HTTP (curl/przeglądarka)

#### 3. Uruchomienie i podpięcie wolumenu
- Utworzenie wolumenu 10GB w panelu OVH
- Przypięcie wolumenu do instancji utworzonej w poprzednim ćwiczeniu
- Odnalezienie wolumenu na liście dostępnych na naszej instancji:
```
lsblk
```
- Dla **nowych** wolumenów konieczne jest utworzenie systemu plików:
```
sudo mkfs.ext4 /dev/sdX
```
- Po utworzeniu systemu plików wolumen może zostać zamontowany w wybranej lokalizacji:
```
sudo mkdir /mnt/demo-volume
sudo mount /dev/sdX /mnt/demo-volume
sudo echo "<h1>Hi!</h1>" > /mnt/demo-volume/index.html
```

#### 4. Kilka instancji w różnych regionach połączonych jedną siecią prywatną (vRack)
- Skonfigurowanie sieci prywatnej w managerze OVH
- Uruchomienie kilku instancji w wybranym regionie np WAW1
- Uruchomienie kilku instancji w wybranym regionie np GRA9
- Upewnienie się, że mamy łączność pomiędzy maszynami wirtualnymi (ping)
- Sprawdznie połączenia pomiędzy węzłami sieci publicznej a prywatnej (mtr)

```
sudo dhclient ens7
```

#### 5. Horizon: zmiana ustawień security group (20 mins)
- Utworzenie użytkownika Public Cloud w managerze
- Użycie utworzonych danych użytkownika do zalogowanie w Horizonie
- Sprawdzenie obcnych ustawień security group.
- Zablokowanie łączności z zewnątrz do maszyn wirtualnych w wybranym regionie, np WAW1 (z wyjątkiem SSH):
    * Usunięcie głównej reguły dla IPv4 (Ingress), która zezwala na pełną łączność (w tym momencie łączność po SSH zostanie przerwana)
    * Dodanie reguły, która pozwoli na łączność po protokole SSH (Ingress IPv4, TCP, port 22)
    * Sprawdzenie łączności
- [demo]Egress - łączność do internetu z maszyny wirtualnej

#### [demo] 6. Openstack CLI: 'Zamknięcie' instancji aby uniemożliwić jej usunięcie.
- Instalacja klienta openstack'a
```
pip3 install python-openstackclient
```
- Pobranie pliku openrc.sh z panelu OVH:
```
source openrc.sh
openstack server list
openstack flavor list
openstack volume list
```
- 'Zamknięcie' instancji:
```
openstack server lock [VM_ID]
```
- Upewnienie się, że maszyna wirtualna nie może zostać usunięcia z poziomu panelu OVH
- Zwolnienie zamknięcia na wybranej instancji:
```
openstack server unlock [VM_ID]
```
- Usunięcie instancji poprzez panel.

#### 7. Managed k8s
- Uruchomienie możliwie małego klaster Kubernetesa (3 węzły)
- Instalacja kubectl'a:
    - Instrukcja instalacji: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
- Połączenie się z klastrem:
    - Skopiowanie pliku kubeconfig pobranego z panelu OVH:
    ```scp -i key kubeconfig.yml ubuntu@[PUBLIC_IP]:~/ ```
    ```export KUBECONFIG=kubeconfig.yml```
- Wylistowanie węzłów klastra:
    ```kubectl get nodes```
- Uruchomienie demonstracyjnego POD'a:
    ```
    kubectl run demo-nginx --image=nginx --port 80
    ```
- Wystawienie POD'a do internetu:
    ```
    kubectl expose pod demo-nginx --type=LoadBalancer
    ```
- Sprawdzenie z przeglądarki adresu IP load balancera

#### 8. Object storage (S3)
- Utworzenie publicznego bucketu S3 w OVHcloud
- Wysłanie pliku
- Pobranie pliku
- Wytłumaczenie pozostałych konfiguracji bucketów S3 oferowanych w OVHcloud
- [demo] Połączenie do bucketa z poziomu klienta:
    - Dodanie użytkownika (object storage operator)
    - Wygenerowanie danych logowania do S3 serwera
    - Instalacja s3 clienta np minio client:
    ```
    wget https://dl.min.io/client/mc/release/linux-amd64/mc
    chmod +x mc
    ```
    - Konfiguracja aliasu do połączenia:
    ```
    mc alias set ovh-public-bucket https://storage.waw.cloud.ovh.net/ ACCESS_KEY SECRET_KEY
    ```
    Do some commands in your bucket:
    - `mc ls ovh-public-bucket/`
    - `mc ls ovh-public-bucket/public-mz-check/`
    - `mc cp ovh-public-bucket/public-mz-check/cv.pdf .`
    - `mc cp something.pdf ovh-public-bucket/public-mz-check/`
