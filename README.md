# AfricanFalls-walkthrough

## Narzędzia

1. FTK Imager
2. Autopsy
3. Mimikatz

## Weryfikacja integralności

Przed przystąpieniem do pracy nad dyskiem zawsze powinniśmy najpierw sprawdzić jego sumę kontrolną oraz to czy zgadza się ona z oficjalną podaną w dokumentacji sprawy. Odczytujemy z pliku `DiskDigger.ad1.txt`, że suma kontrola MD5 wynosi: 9471e69c95d8909ae60ddff30d50ffa1 [Q1] i jest zgodna z oficjalną.


![md5checksum](https://github.com/user-attachments/assets/c4837fb7-e686-4028-927e-128b1bd31277)


## Przygotowanie środowiska

Aby obraz dysku stał się dostępny dla narzędzi analitycznych tj. Autopsy, należy zamontować plik obrazu `DiskDigger.ad1` jako wirtualny dysk w systemie operacyjnym. Idealnym narzędziem do tego zadania jest **FTK Imager**. Uruchamiamy, więc to narzędzie. Dodajemy obraz do zamontowania i wybieramy metodę "*File System / Read Only*":

<img width="1357" height="837" alt="image" src="https://github.com/user-attachments/assets/2612413a-3a7a-4eb7-b04b-22777e938970" />

## Analiza dysku z użyciem Autopsy

Po prawidłowym zainstalowaniu dysku można przejść do jego analizy. Uruchamiamy Autopsy. Wybieramy "*New Case*" oraz nasz nowo zamontowany dysk. Następnie w zakładce "*Select Data Source*" wybieramy "*Local files and folders*' oraz zaznaczamy wszystkie *Timestampy* do uwzględnienia:

<img width="852" height="542" alt="image" src="https://github.com/user-attachments/assets/c3ae9449-48ec-42e8-b72e-90c2fb987eab" />

Przy wyborze należy zaznaczyć wszystkie "*Ingest Modules*", tak aby Autopsy dokonał pełnej korelacji artefaktów. Po kilku minutach możemy przejść do analizy.

Na początek przyjrzyjmy się podstawowym informacjom o systemie i właścicielu urządzenia. Z zakładki "Operation System Information" dowiadujemy się, że nazwa urządzenia to: `DESKTOP-0JS8C2` a systemem operacyjnym jest "Windows 10 Home". Właścicielem jest John Doe:

![os_info](https://github.com/user-attachments/assets/b1750c27-d566-41aa-a235-9f293579180b)


Z kolei z zakładki "OS Accounts" wyczytujemy datę utworzenia konta: 25 kwietnia 2021 r. 19\:57:17 CEST oraz pierwszą ciekawą informację. Mianowice, że na każde pytanie pomocnicze John Doe miał ustawioną tą samą odpowiedź: Roger. Świadczy to o niskim poziomie zabezpieczeń do konta podejrzanego:

![os_accounts](https://github.com/user-attachments/assets/14bb07e1-e6fc-4ba1-82c9-9c86e1890184)

Badanie historii wyszukiwania przeglądarki wykazało, że użytkownik poszukiwał informacji związanych z łamaniem haseł (fraza: `password cracking list` [Q2] 20\:17:38 CEST). Gdy przejdziemy do zakładki "Web History" to zauważymy m.in. adres e-mail w serwise ProtonMail, na który John się zalogował (dreammaker82[@]protonmail[.]com [Q6]):

![email_address](https://github.com/user-attachments/assets/2e3da1df-eb32-4346-ad02-8bb3ce79b1a9)

Następnie w uruchomionych programach ("Run Programs") analizując pliki Prefetch dostrzec można uruchomienie instalatora przeglądarki *Tor Browser* 20\:22:32 CEST. Czas inicjalizacji zbiega się z namiętnym przeszukiwaniem informacji o łamaniu haseł w przeglądarce internetowej. Jednak warto zaznaczyć, że John Doe nigdy nie uruchomił samej przeglądarki Tor, gdyż nie widnieje to w tej zakładce [Q5]:

![tor_activity](https://github.com/user-attachments/assets/e4831123-78d9-43b3-a7f1-c94c1295196a)

W sekcji "File Views" możemy przeszukiwać pliki ze względu na ich rozszerzenie lub typ MIME. Gdy spojrzymy na pliki tekstowe to dostzeżemy tam "ConsoleHost_history.txt", czyli dokument, w którym przechowywana jest historia koemd wywołanych przez użytkownika. Widzimy tam chociażby polecenie "sdelete" służące do trwałego usunięcia danych, ale także "nmap dfir.science" [Q7], a więc wiemy, że użytkownik skanował domenę o tej nazwie:

![nmap](https://github.com/user-attachments/assets/f9b3144a-42cc-4d2e-a477-21a48e86f802)

Następnie przejdźmy do plików "By MIME Type" -> "text" -> "xml". Tam znajduje się m.in. plik "recentservers.xml". Jest to plik generowany przez popularne narzędzie **FileZilla**. Informacje jakie możemy z niego odczytać to adres IP (192.168.1.20 [Q3]), do którego łączył się nasz użytkownik, numer portu 21 (FTP) oraz nazwa użytkownika ('kali'):

![ip_address](https://github.com/user-attachments/assets/46f19055-6c68-4183-b1cb-3350b677dc6f)

Kolejnym istotnym punktem jest sprawdzenie kosza systemowego ("Recycle Bin"). Dostarcza on bowiem kluczowe informacje o tym czym podejrzany nie chciałby się chwalić. Tam widnieje tylko jeden plik, który zawiera istotne dane. Jest to bowiem lista znanych, prostych haseł służąca do ataków typu *brute-force*. Użytkownik skasował plik 29 kwietnia 2021 r. o godzinie 20\:22:17 CEST (18:22:17 UTC [Q4]):

![password_list](https://github.com/user-attachments/assets/ae49aaec-5d49-493c-8e73-906a03196f01)

W kolejnym kroku sprawdzimy zdjęcia znajdujące się na dysku. W tym celu należy przejść do zakładki "Images" w rodzaju plików "By extansions". Dostrzegamy tam chociażby zdjęcie 20210429_152043.jpg, które w metadanych zawiera współrzędne geograficzne (16 S i 23 E), z którego zdjęcie zostało wykonane. Poza tym zawiera czas zrobienia zdjęcia (29 kwietnia 2021 r. 17:20:43 CEST), a także model urządzenia, z którego zostąło dokonane. Odczytujemy **LG-Q725K**:

![20210429_151535](https://github.com/user-attachments/assets/5ad32d0e-73f9-4b4a-8588-fae6670a470d)

Po wprowadzeniu tych koordynatów w serwisie geolokalizacyjnym dostrzegamy nazwę państwa, z którego zostało wykonane zdjęcie - **Zambia** [Q8]

<img width="945" height="465" alt="image" src="https://github.com/user-attachments/assets/fbd98af3-c938-401d-a81b-48fcf1e847df" />

Informację o urządzeniu, z którego zostało wykonane zdjęci możemy wykorzystać do znalezienia pozostałych informacji, m.in. do jakiego folderu trafiały zdjęcia. Przechodzimy więc do zakładki "Shell Bags", a więc kluczy rejestru, gdzie dostrzegamy ścieżkę zawierającą nazwę telefonu oraz docelowy folder, z którego użytkownik kopiował zdjęcia na swój komputer (\Camera [Q9]):

![Camera_dict](https://github.com/user-attachments/assets/1d8b9e89-7804-46ab-8ffc-82f3e8c8b140)


## Odzyskiwanie haseł

Przy użyciu narzędzia Mimikatz, z plików rejestru SAM i SYSTEM wyeksportujmy hashe dla kont lokalnych. Uzyskujemy hash NTLM użytkownika Johna Doe, który po wpisaniu do serwisu hashes.com okazuje się być prostym ciągiem znaków: **ctf2021** [Q11]:


<img width="1346" height="772" alt="image" src="https://github.com/user-attachments/assets/611ce97b-65c4-4710-8cff-cfe40ff8898f" />


Korzystając z hashes.com możemy złamać również hash dla użytkownika Anon, które okazuje się być również bardzo proste (AFR1CA!) [Q10]:


<img width="639" height="330" alt="image" src="https://github.com/user-attachments/assets/d50c4dd3-6e1c-46e5-b2b4-447e13c421eb" />















