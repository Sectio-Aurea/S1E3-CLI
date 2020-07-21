### EPISODE 3 - MIGRACJA APLIKACJI DO CHMURY IBM - CLI
1. Opis procedury instalacji narzędzia `ibmcloud` (a.k.a `ic`) dostępna jest [tutaj](https://cloud.ibm.com/docs/cli).
2. Utworzenie klastra *Kubernetes* z wykorzystaniem serwisu *IBM Kubernetes Service* w chmurze publicznej IBM.

   1. Zalogowanie użytkownika w regionie eu-de wykorzystując API endpoint *cloud.ibm.com*.
   ```
   ibmcloud login -a cloud.ibm.com -r eu-de
   ```
   
   2. Odczyt informacji o dostępnych lokalizacjach (strefach) w tym interesującej nas *fra02*.
   ```
   ibmcloud ks locations
   ```
   
   3. Odczyt informacji o dostępnych konfiguracjach sprzętowych węzłów klastra włączając interesującą nas konfigurację *u2c.2x4*.
   ```
   ibmcloud ks flavors --zone fra02
   ```
   
   4. Utworzenie klastra zbudowanego z trzech węzłów 
   ```
   ibmcloud ks cluster create classic --name myOwnCLusterName --zone fra02 --flavor u2c.2x4 --hardware shared --workers 3
   ```
   **Uwaga!**
   Komenda przedstawiona powyżej automatycznie stworzy w infrastrukturze dwa VLAN-y: publiczny i prywatny, jakie wykorzystane są do komunikacji z klastrem. 
   W przypadku potrzeby stworzenia drugiego klastra K8s lub w przypadku, gdy pierwszy został usunięty i jest potrzeba utworzenia go ponownie, 
   wymagane jest, aby w komendzie `ibmcloud ks cluster create` podać identyfikatory VLAN-u publicznego i prywatnego, odpowiednio po parametrach: 
   `--private-vlan` oraz `--public-vlan`. Identyfikatory VLAN-ów odczytujemy wydając komendę `ibmcloud ks vlan ls --zone fra02`.
   
   5. Sprawdzenie statusu działania klastra.
   ```
   ibmcloud ks cluster ls
   ```
   
   6. Konfiguracja narzędzia `kubectl` 
   ```
   ibmcloud ks cluster config -c myOwnCLusterName
   ```

3. Konfiguracja repozytoium obrazów *Container Registry* i skopiowanie przykładowego obrazu z *Docker Hub*

   1. Sprawdzenie konfiguracji tzw. *Secrets* w konfiguracji klastra.
   ```
   kubectl get secrets -n default | grep "icr-io"
   ```
   
   2. Wskazanie regionu w jakim chcemy skonfigurować *Container Registry*.
   ```
   ibmcloud cr region-set eu-de
   ```
   
   3. Utworzenie nowej przestrzeni nazw.
   ```
   ibmcloud cr namespace-add prodns
   ```
   
   4. Uwierzytelnienie lokalnego Docker-a w *IBM Container Registry*
   ```
   ibmcloud cr login
   ```
   
   5. Pobranie przykładowego obrazu *hello-world* na lokalną maszynę.
   ```
   docker pull hello-world
   ```
   
   6. Otagowanie pobranego obrazu
   ```
   docker tag hello-world de.icr.io/prodns/hello-world>:latest
   ```
   
   7. Wgranie otagowanego obrazu do repozytorium *IBM Container Registry*
   ```
   docker push de.icr.io/prodns/hello-world>:latest
   ```
   
   8. Odczyt listy wgranych obrazów.
   ```
   ibmcloud cr image-list
   ```
 
