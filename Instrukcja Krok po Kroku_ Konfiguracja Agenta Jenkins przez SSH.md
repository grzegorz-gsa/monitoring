# Instrukcja Krok po Kroku: Konfiguracja Agenta Jenkins przez SSH

Ten dokument zawiera szczegółową instrukcję z konkretnymi komendami do wykonania na każdym serwerze. Zakładamy, że masz dwa serwery Ubuntu - jeden z zainstalowanym Jenkinsem (master) i drugi, który będzie agentem.

## Informacje o Serwerach

Przed rozpoczęciem, przygotuj następujące informacje:

- **Serwer Jenkins Master:**
  - Adres IP: `<MASTER_IP>` (np. 192.168.1.10)
  - Użytkownik: `ubuntu` (lub inny użytkownik z uprawnieniami sudo)
  
- **Serwer Jenkins Agent:**
  - Adres IP: `<AGENT_IP>` (np. 192.168.1.20)
  - Użytkownik: `ubuntu` (lub inny użytkownik z uprawnieniami sudo)

---

## CZĘŚĆ 1: Przygotowanie Serwera Agenta

### Krok 1.1: Połącz się z serwerem agenta

**Na swoim komputerze lokalnym** otwórz terminal i połącz się z serwerem agenta:

```bash
ssh ubuntu@<AGENT_IP>
```

Przykład:
```bash
ssh ubuntu@192.168.1.20
```

### Krok 1.2: Zaktualizuj system i zainstaluj niezbędne pakiety

**Na serwerze agenta** wykonaj następujące komendy:

```bash
# Aktualizacja listy pakietów
sudo apt update

# Instalacja Java (wymagana dla agenta Jenkins)
sudo apt install -y openjdk-17-jre

# Instalacja dodatkowych narzędzi (git, python3, etc.)
sudo apt install -y git python3-pip python3-venv

# Sprawdź wersję Java
java -version
```

Powinieneś zobaczyć informację o wersji Java (np. openjdk 17.x.x).

### Krok 1.3: Utwórz dedykowanego użytkownika jenkins

**Na serwerze agenta** wykonaj:

```bash
# Utwórz użytkownika jenkins z katalogiem domowym
sudo useradd -m -s /bin/bash jenkins

# Ustaw hasło dla użytkownika jenkins (opcjonalnie, ale zalecane)
sudo passwd jenkins
```

Wpisz hasło dwa razy (możesz użyć prostego hasła, np. `jenkins123`).

### Krok 1.4: Przełącz się na użytkownika jenkins

**Na serwerze agenta** wykonaj:

```bash
# Przełącz się na użytkownika jenkins
sudo su - jenkins
```

Teraz jesteś zalogowany jako użytkownik `jenkins`. Prompt powinien wyglądać tak: `jenkins@hostname:~$`

### Krok 1.5: Utwórz katalog .ssh i skonfiguruj uprawnienia

**Na serwerze agenta jako użytkownik jenkins** wykonaj:

```bash
# Utwórz katalog .ssh
mkdir -p ~/.ssh

# Ustaw odpowiednie uprawnienia
chmod 700 ~/.ssh

# Utwórz plik authorized_keys
touch ~/.ssh/authorized_keys

# Ustaw uprawnienia dla pliku
chmod 600 ~/.ssh/authorized_keys

# Sprawdź strukturę
ls -la ~/.ssh
```

Powinieneś zobaczyć katalog `.ssh` z uprawnieniami `drwx------` i plik `authorized_keys` z uprawnieniami `-rw-------`.

### Krok 1.6: Wyjdź z użytkownika jenkins

**Na serwerze agenta** wykonaj:

```bash
# Wróć do użytkownika ubuntu
exit
```

**Pozostaw połączenie SSH z serwerem agenta otwarte** - będzie jeszcze potrzebne.

---

## CZĘŚĆ 2: Przygotowanie Serwera Master

### Krok 2.1: Połącz się z serwerem master

**Otwórz nowe okno terminala** na swoim komputerze lokalnym i połącz się z serwerem master:

```bash
ssh ubuntu@<MASTER_IP>
```

Przykład:
```bash
ssh ubuntu@192.168.1.10
```

### Krok 2.2: Znajdź użytkownika, jako który działa Jenkins

**Na serwerze master** wykonaj:

```bash
# Sprawdź, jako który użytkownik działa proces Jenkins
ps aux | grep jenkins
```

Zazwyczaj będzie to użytkownik `jenkins`. Jeśli Jenkins działa jako inny użytkownik, użyj tej nazwy w kolejnych krokach.

### Krok 2.3: Przełącz się na użytkownika jenkins

**Na serwerze master** wykonaj:

```bash
# Przełącz się na użytkownika jenkins
sudo su - jenkins
```

### Krok 2.4: Sprawdź, czy istnieją klucze SSH

**Na serwerze master jako użytkownik jenkins** wykonaj:

```bash
# Sprawdź, czy istnieją klucze SSH
ls -la ~/.ssh/
```

Jeśli widzisz pliki `id_rsa` i `id_rsa.pub`, klucze już istnieją. Jeśli nie, przejdź do następnego kroku.

### Krok 2.5: Wygeneruj parę kluczy SSH (jeśli nie istnieją)

**Na serwerze master jako użytkownik jenkins** wykonaj:

```bash
# Wygeneruj klucze SSH (bez hasła)
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```

Klucze zostaną utworzone w katalogu `~/.ssh/`.

### Krok 2.6: Wyświetl klucz publiczny

**Na serwerze master jako użytkownik jenkins** wykonaj:

```bash
# Wyświetl klucz publiczny
cat ~/.ssh/id_rsa.pub
```

Skopiuj **całą** zawartość klucza (zaczyna się od `ssh-rsa` i kończy na `jenkins@hostname`). Będzie wyglądać mniej więcej tak:

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDExampleKeyContent... jenkins@master-hostname
```

**Skopiuj ten klucz do schowka** - będzie potrzebny w następnym kroku.

---

## CZĘŚĆ 3: Konfiguracja Autoryzacji SSH

### Krok 3.1: Dodaj klucz publiczny na serwerze agenta

**Wróć do okna terminala z połączeniem do serwera agenta** (jako użytkownik `ubuntu`).

**Na serwerze agenta** wykonaj:

```bash
# Przełącz się na użytkownika jenkins
sudo su - jenkins

# Otwórz plik authorized_keys w edytorze
nano ~/.ssh/authorized_keys
```

**W edytorze nano:**
1. Wklej skopiowany wcześniej klucz publiczny z serwera master
2. Naciśnij `Ctrl+O` (zapisz)
3. Naciśnij `Enter` (potwierdź nazwę pliku)
4. Naciśnij `Ctrl+X` (wyjdź)

**Alternatywnie**, jeśli masz klucz w schowku, możesz użyć jednej komendy:

```bash
# Dodaj klucz do authorized_keys (zastąp <KLUCZ_PUBLICZNY> faktycznym kluczem)
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDExampleKeyContent... jenkins@master-hostname" >> ~/.ssh/authorized_keys
```

### Krok 3.2: Sprawdź zawartość pliku authorized_keys

**Na serwerze agenta jako użytkownik jenkins** wykonaj:

```bash
# Wyświetl zawartość pliku
cat ~/.ssh/authorized_keys
```

Powinieneś zobaczyć dodany klucz publiczny.

### Krok 3.3: Wyjdź z użytkownika jenkins na agencie

**Na serwerze agenta** wykonaj:

```bash
# Wróć do użytkownika ubuntu
exit
```

---

## CZĘŚĆ 4: Test Połączenia SSH

### Krok 4.1: Przetestuj połączenie z mastera do agenta

**Wróć do okna terminala z serwerem master** (powinieneś być zalogowany jako użytkownik `jenkins`).

**Na serwerze master jako użytkownik jenkins** wykonaj:

```bash
# Przetestuj połączenie SSH do agenta
ssh jenkins@<AGENT_IP>
```

Przykład:
```bash
ssh jenkins@192.168.1.20
```

**Przy pierwszym połączeniu** zobaczysz komunikat:

```
The authenticity of host '192.168.1.20 (192.168.1.20)' can't be established.
ECDSA key fingerprint is SHA256:...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Wpisz `yes` i naciśnij Enter.

**Jeśli wszystko jest poprawnie skonfigurowane**, zostaniesz zalogowany na serwer agenta jako użytkownik `jenkins` **bez podawania hasła**.

### Krok 4.2: Sprawdź połączenie

**Na serwerze agenta (po połączeniu SSH)** wykonaj:

```bash
# Sprawdź, gdzie jesteś
pwd

# Sprawdź, kim jesteś
whoami

# Sprawdź wersję Java
java -version
```

Powinieneś zobaczyć:
- Katalog domowy: `/home/jenkins`
- Użytkownik: `jenkins`
- Wersja Java: `openjdk 17.x.x`

### Krok 4.3: Wyjdź z połączenia testowego

**Na serwerze agenta** wykonaj:

```bash
# Wyjdź z połączenia SSH
exit
```

Wrócisz do serwera master jako użytkownik `jenkins`.

### Krok 4.4: Pobierz klucz prywatny do konfiguracji Jenkins

**Na serwerze master jako użytkownik jenkins** wykonaj:

```bash
# Wyświetl klucz prywatny
cat ~/.ssh/id_rsa
```

Skopiuj **całą** zawartość klucza prywatnego (od `-----BEGIN OPENSSH PRIVATE KEY-----` do `-----END OPENSSH PRIVATE KEY-----` włącznie). **Będzie to potrzebne w interfejsie Jenkins**.

### Krok 4.5: Wyjdź z użytkownika jenkins na masterze

**Na serwerze master** wykonaj:

```bash
# Wróć do użytkownika ubuntu
exit
```

---

## CZĘŚĆ 5: Konfiguracja Węzła w Interfejsie Jenkins

### Krok 5.1: Zaloguj się do Jenkins

Otwórz przeglądarkę i przejdź do interfejsu Jenkins:

```
http://<MASTER_IP>:8080
```

Zaloguj się swoimi danymi.

### Krok 5.2: Przejdź do zarządzania węzłami

1. Kliknij **Dashboard** (logo Jenkins w lewym górnym rogu)
2. Kliknij **Manage Jenkins** (w menu po lewej stronie)
3. Kliknij **Nodes** (lub **Manage Nodes and Clouds**)

### Krok 5.3: Dodaj nowy węzeł

1. Kliknij **New Node** (w menu po lewej stronie)
2. W polu **Node name** wpisz nazwę, np. `ubuntu-agent-01`
3. Zaznacz **Permanent Agent**
4. Kliknij **Create**

### Krok 5.4: Skonfiguruj węzeł - Podstawowe ustawienia

Na stronie konfiguracji węzła wypełnij pola:

**Sekcja General:**
- **Name:** `ubuntu-agent-01` (już wypełnione)
- **Description:** `Ubuntu agent for pytest jobs` (opcjonalnie)
- **Number of executors:** `2` (liczba równoległych zadań)
- **Remote root directory:** `/home/jenkins` (katalog roboczy na agencie)
- **Labels:** `ubuntu python pytest` (etykiety oddzielone spacjami)
- **Usage:** Wybierz **Use this node as much as possible**

### Krok 5.5: Skonfiguruj węzeł - Metoda uruchamiania

**Sekcja Launch method:**
- Wybierz **Launch agents via SSH** z listy rozwijanej

Po wybraniu pojawią się dodatkowe pola:

- **Host:** Wpisz adres IP agenta, np. `192.168.1.20` lub `<AGENT_IP>`
- **Credentials:** Kliknij przycisk **Add** obok pola, a następnie wybierz **Jenkins**

### Krok 5.6: Dodaj poświadczenia SSH

W oknie **Jenkins Credentials Provider** wypełnij:

- **Domain:** Global credentials (unrestricted)
- **Kind:** Wybierz **SSH Username with private key**
- **Scope:** Global
- **ID:** `jenkins-agent-ssh-key` (unikalny identyfikator)
- **Description:** `SSH key for Jenkins agent` (opcjonalnie)
- **Username:** `jenkins`
- **Private Key:** Zaznacz **Enter directly**
  - Kliknij przycisk **Add**
  - W polu tekstowym wklej **cały klucz prywatny** skopiowany wcześniej z serwera master (od `-----BEGIN OPENSSH PRIVATE KEY-----` do `-----END OPENSSH PRIVATE KEY-----`)
- **Passphrase:** Pozostaw puste (jeśli nie ustawiłeś hasła dla klucza)

Kliknij **Add**.

### Krok 5.7: Wybierz poświadczenia i skonfiguruj weryfikację

Wróć do konfiguracji węzła:

- **Credentials:** Z listy rozwijanej wybierz właśnie dodane poświadczenia: `jenkins (SSH key for Jenkins agent)`
- **Host Key Verification Strategy:** Wybierz **Non-verifying Verification Strategy**

**Uwaga:** W środowisku produkcyjnym zaleca się użycie **Known hosts file Verification Strategy**, ale dla uproszczenia konfiguracji testowej używamy **Non-verifying**.

### Krok 5.8: Pozostałe ustawienia

**Sekcja Availability:**
- Wybierz **Keep this agent online as much as possible**

**Pozostałe sekcje** możesz pozostawić z domyślnymi wartościami.

### Krok 5.9: Zapisz konfigurację

Kliknij **Save** na dole strony.

---

## CZĘŚĆ 6: Uruchomienie i Weryfikacja Agenta

### Krok 6.1: Sprawdź status agenta

Po zapisaniu zostaniesz przeniesiony do listy węzłów. Powinieneś zobaczyć:

- **Built-In Node** (master)
- **ubuntu-agent-01** (twój nowy agent)

Agent może być w jednym z następujących stanów:
- **Czerwona ikona** (offline) - agent nie jest połączony
- **Niebieska ikona** (online) - agent jest połączony i gotowy

### Krok 6.2: Uruchom agenta (jeśli jest offline)

Jeśli agent jest offline:

1. Kliknij na nazwę agenta **ubuntu-agent-01**
2. W menu po lewej stronie kliknij **Log**
3. Sprawdź logi - zobaczysz tam szczegóły próby połączenia

Jeśli widzisz błędy, najczęstsze przyczyny to:
- Nieprawidłowy klucz prywatny
- Błędny adres IP agenta
- Brak dostępu sieciowego między serwerami
- Brak Javy na agencie

### Krok 6.3: Weryfikacja połączenia

Jeśli agent jest **online** (niebieska ikona), gratulacje! Agent jest poprawnie skonfigurowany.

Możesz to zweryfikować:

1. Kliknij na nazwę agenta **ubuntu-agent-01**
2. Powinieneś zobaczyć:
   - **Status:** Agent is connected
   - **Free Disk Space:** Informacje o wolnym miejscu na dysku agenta
   - **Response Time:** Czas odpowiedzi

---

## CZĘŚĆ 7: Test Działania Agenta

### Krok 7.1: Utwórz testowy job

1. Wróć do **Dashboard**
2. Kliknij **New Item**
3. Wpisz nazwę: `Test-Agent-Connection`
4. Wybierz **Freestyle project**
5. Kliknij **OK**

### Krok 7.2: Skonfiguruj testowy job

Na stronie konfiguracji:

1. Zaznacz **Restrict where this project can be run**
2. W polu **Label Expression** wpisz: `ubuntu`
3. Przewiń do sekcji **Build Steps**
4. Kliknij **Add build step** > **Execute shell**
5. W polu **Command** wpisz:

```bash
#!/bin/bash
echo "========================================="
echo "Test połączenia z agentem Jenkins"
echo "========================================="
echo ""
echo "Hostname: $(hostname)"
echo "Username: $(whoami)"
echo "Working directory: $(pwd)"
echo "Java version:"
java -version
echo ""
echo "Python version:"
python3 --version
echo ""
echo "========================================="
echo "Test zakończony pomyślnie!"
echo "========================================="
```

6. Kliknij **Save**

### Krok 7.3: Uruchom testowy job

1. Kliknij **Build Now** w menu po lewej stronie
2. W sekcji **Build History** pojawi się nowy build (np. #1)
3. Kliknij na numer buildu
4. Kliknij **Console Output**

Powinieneś zobaczyć output z informacjami o agencie, w tym:
- Hostname agenta
- Użytkownik: `jenkins`
- Wersje Java i Pythona

Jeśli widzisz komunikat **"Test zakończony pomyślnie!"**, agent działa poprawnie!

---

## Rozwiązywanie Problemów

### Problem: Agent nie może się połączyć

**Sprawdź logi:**
1. Przejdź do **Manage Jenkins** > **Nodes**
2. Kliknij na nazwę agenta
3. Kliknij **Log**

**Najczęstsze błędy:**

**Błąd: "Permission denied (publickey)"**
- Sprawdź, czy klucz publiczny jest poprawnie dodany do `~/.ssh/authorized_keys` na agencie
- Sprawdź uprawnienia: katalog `.ssh` powinien mieć `700`, plik `authorized_keys` powinien mieć `600`

**Błąd: "Connection refused"**
- Sprawdź, czy serwer SSH działa na agencie: `sudo systemctl status ssh`
- Sprawdź, czy firewall nie blokuje portu 22

**Błąd: "java: command not found"**
- Zainstaluj Javę na agencie: `sudo apt install -y openjdk-17-jre`

### Problem: Nie mogę skopiować klucza prywatnego

**Na serwerze master jako użytkownik jenkins:**

```bash
# Wyświetl klucz prywatny i zapisz do pliku
cat ~/.ssh/id_rsa > /tmp/jenkins_private_key.txt

# Zmień uprawnienia, aby można było go pobrać
chmod 644 /tmp/jenkins_private_key.txt
```

Następnie możesz pobrać plik `/tmp/jenkins_private_key.txt` i skopiować jego zawartość do interfejsu Jenkins.

**Pamiętaj o usunięciu pliku po skopiowaniu:**

```bash
rm /tmp/jenkins_private_key.txt
```

---

## Podsumowanie Wykonanych Kroków

✅ Zainstalowano Javę i narzędzia na serwerze agenta  
✅ Utworzono użytkownika `jenkins` na agencie  
✅ Skonfigurowano katalog `.ssh` i plik `authorized_keys` na agencie  
✅ Wygenerowano parę kluczy SSH na serwerze master  
✅ Skopiowano klucz publiczny z mastera do agenta  
✅ Przetestowano połączenie SSH  
✅ Dodano węzeł w interfejsie Jenkins  
✅ Skonfigurowano poświadczenia SSH  
✅ Zweryfikowano działanie agenta testowym jobem  

**Agent Jenkins jest teraz gotowy do wykonywania zadań!**

