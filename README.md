# Programowanie Full-Stack w Chmurze Obliczeniowej

      Marek Prokopiuk
      grupa dziekańska: 2.3
      numer albumu: 097710
## Zadanie nr 1
<p align="justify">
Przedstawione zostało rozwiązanie zadania nr 1 z laboratorium <i>Programowanie Full-Stack w Chmurze Obliczeniowej</i>. Zrealizowane zostały zarówno część obowiązkowa, jak i nieobowiązkowa zadania. Poniższe sprawozdanie z wykonania zadania zostało podzielone na podpunkty zgodnie z numeracją w treści zadania (A, B itd.).
</p>

---

### A. Utworzenie klastra z węzłem głównym oraz trzema węzłami roboczymi
<p align="justify">
Na początku należało odpowiednio utworzyć klaster. Zostało to zrealizowane z wykorzystaniem sterownika Docker oraz wtyczki CNI <b>Calico</b>. Użyto konfiguracji obejmującej cztery węzły: jeden główny oraz trzy węzły robocze. Po uruchomieniu klastra zweryfikowano dostępność węzłów oraz nadano im odpowiednie etykiety (A, B, C). Zostały one następnie wykorzystane w mechanizmie <b>nodeAffinity</b> do sterowania rozmieszczeniem Pod-ów pomiędzy węzłami. Klaster utworzono z wykorzystaniem polecenia:
</p>

```diff
minikube start -n 4 --cni='calico' -d docker --cpus=2 --memory=2g
```
<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/A1.png" style="width: 80%; height: 80%" /></p>
<p align="center">
  <i>Rys. 1. Uruchomienie klastra Minikube z CNI Calico (fragment)</i>
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/A2.png" style="width: 80%; height: 80%" /></p>
<p align="center">
  <i>Rys. 2. Weryfikacja poprawności węzłów oraz nadanie im etykiet A, B, C</i>
</p>

---

### B. Utworzenie zestawu manifestów opisujących określone obiekty środowiska Kubernetes
<p align="justify">
W kolejnym etapie zadania należało utworzyć zestaw manifestów YAML opisujących wymagane obiekty środowiska Kubernetes w dwóch różnych przestrzeniach nazw: <b>frontend</b> oraz <b>backend</b>. Rozdzielenie aplikacji na osobne <i>namespace’y</i> umożliwia niezależne zarządzanie zasobami, politykami sieciowymi oraz limitami zasobów w dalszych etapach zadania. Na początku utworzono dwie wspomniane wcześniej przestrzenie nazw, a następnie zweryfikowano ich poprawne utworzenie.</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/B1.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 3. Utworzenie przestrzeni nazw frontend oraz backend</i>
</p>

<p align="justify">
W przestrzeni nazw <b>frontend</b> utworzono obiekt Deployment o nazwie <b>frontend</b>, na bazie obrazu <i>nginx</i>, z trzema replikami Pod-ów. Zastosowano mechanizm <b>nodeAffinity</b>, który wymusza uruchamianie Pod-ów <i>frontend</i> wyłącznie na węzłach oznaczonych etykietami A oraz C. Dzięki temu zapewniono, że Pod-y tego Deployment-u znajdą się na dowolnym węźle z dozwolonych węzłów, ale nie na tym, na którym będą działać Pod-y <i>backend</i> oraz <i>my-sql</i> (węzeł B).
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/B2.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 4. Manifest Deployment-u frontend z konfiguracją nodeAffinity (frontend.yaml)</i>
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/B3.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 5. Wdrożenie Deployment-u frontend oraz widoczne rozmieszczenie Pod-ów na węzłach A i C</i>
</p>

<p align="justify">
W przestrzeni nazw <b>backend</b> utworzono Deployment o nazwie <b>backend</b>, również oparty o obraz <i>nginx</i>, ale z jedną repliką. W tym przypadku również zastosowano <b>nodeAffinity</b>, jednak z ograniczeniem do węzła oznaczonego etykietą B. Zapewnia to, że Pod backend zostanie uruchomiony na tym węźle, tak samo jak utworzony później Pod <i>my-sql</i>.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/B4.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 6. Manifest Deployment-u backend (backend.yaml)</i>
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/B5.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 7. Uruchomienie Deployment-u backend na węźle B</i>
</p>

<p align="justify">
W przestrzeni nazw <b>backend</b> utworzono również Pod o nazwie <b>my-sql</b>, oparty o obraz <i>mysql</i>. Pod ten został przypisany do tego samego węzła klastra co Pod-y Deployment-u <i>backend</i> (zgodnie z treścią polecenia) poprzez użycie mechanizmu <b>nodeAffinity</b>. Dodatkowo skonfigurowano zmienną środowiskową <i>MYSQL_ROOT_PASSWORD</i>, niezbędną do poprawnego uruchomienia kontenera MySQL.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/B6.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 8. Manifest Pod-a my-sql (my-sql.yaml)</i>
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/B7.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 9. Widoczne wspólne umiejscowienie Pod-ów backend oraz my-sql na węźle B</i>
</p>

<p align="justify">
Dla zapewnienia komunikacji sieciowej pomiędzy komponentami aplikacji utworzono odpowiednie obiekty <i>Service</i> zgodnie z treścią zadania. Dla Deployment-u <i>frontend</i> zastosowano Service typu <b>NodePort</b>, umożliwiający dostęp do aplikacji z zewnątrz klastra. Natomiast dla Deployment-u <i>backend</i> oraz Pod-a <i>my-sql</i> utworzono usługi typu <b>ClusterIP</b>, ponieważ komponenty te powinny być dostępne wyłącznie wewnątrz klastra. Porty usług zostały dobrane samodzielnie, natomiast parametr <i>targetPort</i> odpowiada rzeczywistym portom kontenerów. W przypadku <b>NodePort</b> nie wskazano konkretnego numeru portu węzła, pozostawiając Kubernetesowi jego automatyczne przydzielenie.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/B8.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 10. Service NodePort dla Deployment-u frontend (frontend-service.yaml)</i>
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/B9.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 11. Service ClusterIP dla Deployment-u backend (backend-service.yaml)</i>
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/B10.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 12. Service ClusterIP dla Pod-a my-sql (mysql-service.yaml)</i>
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/B11.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 13. Utworzenie obiektów Service i polecenia potwierdzające poprawność wykonania zadania</i>
</p>

<p align="justify">
Poprawność działania usług zweryfikowano poprzez uruchomienie testowego Pod-a z obrazem <i>busybox</i> oraz wykonanie połączenia do usługi <i>frontend</i> za pomocą narzędzia <i>wget</i>. Test potwierdził poprawne działanie mechanizmu <i>Service</i> oraz rozwiązywania nazw DNS wewnątrz klastra.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/B12.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 14. Test dostępności usługi frontend z poziomu klastra</i>
</p>

---

### C. Opracowanie polityk sieciowych (<i>NetworkPolicy</i>)
<p align="justify">
W dalszej części zadania należało opracować polityki sieciowe (obiekty <i>NetworkPolicy</i>), które ograniczają komunikację pomiędzy Pod-ami zgodnie z wymaganiami treści polecenia. W środowisku Kubernetes obowiązuje zasada, że w momencie utworzenia co najmniej jednej polityki sieciowej w danej przestrzeni nazw, cały ruch nieobjęty regułami polityki zostaje domyślnie zablokowany (<i>default deny</i>). Oznacza to, że dozwolony jest wyłącznie ruch jawnie zdefiniowany w obiektach <i>NetworkPolicy</i>.
</p>

<p align="justify">
W celu ustalenia możliwej komunikacji pomiędzy Pod-ami znajdującymi się w różnych przestrzeniach nazw wykorzystano mechanizm <b>namespaceSelector</b>. Zgodnie z dokumentacją Kubernetes, selektor ten działa w oparciu o etykiety przypisane do obiektów <i>Namespace</i>. Standardową etykietą identyfikującą przestrzeń nazw jest <i>kubernetes.io/metadata.name</i>. Dla Pod-ów Deployment-u <b>frontend</b> w przestrzeni nazw <b>frontend</b> utworzono politykę sieciową typu <b>Egress</b>, która zezwala wyłącznie na połączenia wychodzące do Pod-ów należących do Deployment-u <b>backend</b> w przestrzeni nazw <b>backend</b> na porcie <b>80/TCP</b>. Jakikolwiek inny ruch wychodzący z Pod-ów frontend (np. do innych Pod-ów, usług czy przestrzeni nazw) jest blokowany.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/C1.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 15. NetworkPolicy ograniczająca ruch wychodzący z Pod-ów frontend (frontend-networkpolicy.yaml)</i>
</p>

<p align="justify">
Dla Pod-ów Deployment-u <b>backend</b> w przestrzeni nazw <b>backend</b> utworzono kolejną politykę sieciową typu <b>Egress</b>. Polityka ta dopuszcza:
<ul>
  <li>połączenia do Pod-a <i>my-sql</i> wyłącznie na porcie <b>3306/TCP</b>,</li>
  <li>połączenia do Pod-ów Deployment-u <i>frontend</i> w przestrzeni nazw <i>frontend</i> na dowolnym porcie.</li>
</ul>
Tym samym spełniono wymagania zadania dotyczące dwukierunkowej komunikacji pomiędzy frontendem i backendem oraz ograniczenia dostępu do MySQL zarówno z backendu, jak i frontendu.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/C2.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 16. NetworkPolicy dla Pod-ów backend z ograniczeniem dostępu do my-sql (backend-networkpolicy.yaml)</i>
</p>

<p align="justify">
Po utworzeniu polityk sieciowych zweryfikowano ich poprawne zastosowanie oraz zakres obowiązywania przy pomocy odpowiednich poleceń <i>kubectl get</i> oraz <i>kubectl describe</i>.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/C3.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 17. Weryfikacja utworzonych obiektów NetworkPolicy</i>
</p>

<p align="justify">
Poprawność działania polityk sieciowych została potwierdzona poprzez wykonanie testów z użyciem testowych Pod-ów uruchomionych na obrazie <i>nicolaka/netshoot</i>. Testy wykazały, że:
<ul>
  <li>Pod-y frontend mogą komunikować się wyłącznie z backendem na dozwolonym porcie,</li>
  <li>nie mają dostępu do usługi my-sql ani do własnej usługi frontend,</li>
  <li>Pod-y backend mogą komunikować się zarówno z frontendem, jak i z my-sql na porcie 3306,</li>
  <li>połączenia niezgodne z polityką są skutecznie blokowane.</li>
</ul>
</p>

<p align="justify">
Dodatkowo podczas testów dla przestrzeni nazw <b>frontend</b> celowo uruchomiono dwa Pod-y testowe, z których tylko jeden posiadał etykietę <i>app=frontend</i> objętą selektorem <i>podSelector</i> w polityce sieciowej. Wyniki testów jednoznacznie potwierdziły, że ograniczenia zdefiniowane w obiekcie <i>NetworkPolicy</i> obowiązują wyłącznie Pod-y spełniające warunki selektora. Pod posiadający odpowiednią etykietę podlegał restrykcjom komunikacyjnym, natomiast drugi Pod, nieobjęty polityką, zachowywał pełną swobodę komunikacji sieciowej. Test ten potwierdził poprawne i selektywne działanie mechanizmu <i>podSelector</i> w politykach sieciowych Kubernetes.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/C4.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 18. Testy polityki sieciowej dla Pod-ów frontend</i>
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/C5.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 19. Testy polityki sieciowej dla Pod-ów backend</i>
</p>

<p align="justify">
Podczas testów wykorzystano bezpośrednie adresy IP usług. Brak możliwości użycia nazw DNS wynika z faktu, że po zdefiniowaniu polityk <i>NetworkPolicy</i> typu <b>Egress</b> cały ruch wychodzący jest domyślnie blokowany, a w zdefiniowanych regułach nie dopuszczono ruchu do usługi <i>kube-dns</i>. Zachowanie to jest zgodne z oczekiwaniami i dodatkowo potwierdza poprawność działania polityk sieciowych.
</p>

---

### D. Ograniczenie liczby Pod-ów oraz zasobów CPU i pamięci RAM
<p align="justify">
W ramach punktu D zastosowano mechanizmy <b>ResourceQuota</b> oraz <b>LimitRange</b> w obu przestrzeniach nazw: <b>frontend</b> oraz <b>backend</b>. Obiekt <i>ResourceQuota</i> został wykorzystany do ograniczenia maksymalnej liczby Pod-ów oraz sumarycznego zużycia zasobów CPU i pamięci RAM w obrębie danej przestrzeni nazw, zgodnie z wymaganiami treści zadania. Z kolei <i>LimitRange</i> wprowadza domyślne wartości <i>requests</i> oraz <i>limits</i> dla pojedynczych kontenerów, co zapobiega tworzeniu Pod-ów bez zdefiniowanych zasobów oraz umożliwia poprawne egzekwowanie polityki quota.
</p>

<p align="justify">
Dla przestrzeni nazw <b>frontend</b> utworzono obiekt <i>ResourceQuota</i>, który ogranicza:
</p>

<ul>
  <li>maksymalną liczbę Pod-ów do <b>10</b>,</li>
  <li>sumaryczne zużycie CPU do <b>1 CPU (1000m)</b>,</li>
  <li>sumaryczne zużycie pamięci RAM do <b>1,5 Gi (1536 Mi)</b>.</li>
</ul>

<p align="justify">
Wartość 1,5 GiB została przeliczona zgodnie z zasadami Kubernetes jako 1,5 × 1024 Mi = 1536 Mi. Ograniczenia te odpowiadają bezpośrednio wymaganiom zadania.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/D1.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 20. ResourceQuota dla przestrzeni nazw frontend (frontend-quota.yaml)</i>
</p>

<p align="justify">
Uzupełniająco zastosowano obiekt <i>LimitRange</i> w przestrzeni nazw <b>frontend</b>, który definiuje domyślne wartości <i>requests</i> oraz <i>limits</i> dla pojedynczych kontenerów na poziomie <b>100m CPU</b> oraz <b>128Mi pamięci RAM</b>. Dzięki temu każdy <b>nowo tworzony Pod</b> posiada zdefiniowane zasoby, nawet jeśli nie zostały one jawnie określone w manifeście Deployment-u.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/D2.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 21. LimitRange dla przestrzeni nazw frontend (frontend-limitrange.yaml)</i>
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/D3.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 22. Zastosowanie ResourceQuota oraz LimitRange w przestrzeni nazw frontend</i>
</p>

<p align="justify">
Zerowe wartości przy CPU i pamięci RAM wynikają z tego, że limity i defaulty egzekwowane są tylko dla nowo tworzonych Pod-ów. Istniejące Pod-y nie są modyfikowane ani zabijane, natomiast nowe Pod-y muszą spełniać zasady i zostaną odrzucone jeśli ich nie spełniają. Analogicznie skonfigurowano ograniczenia dla przestrzeni nazw <b>backend</b>. Utworzony obiekt <i>ResourceQuota</i> ogranicza:
</p>

<ul>
  <li>maksymalną liczbę Pod-ów do <b>3</b>,</li>
  <li>sumaryczne zużycie CPU do <b>1 CPU (1000m)</b>,</li>
  <li>sumaryczne zużycie pamięci RAM do <b>1 Gi</b>.</li>
</ul>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/D4.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 23. ResourceQuota dla przestrzeni nazw backend (backend-quota.yaml)</i>
</p>

<p align="justify">
W przestrzeni nazw <b>backend</b> utworzono obiekt <i>LimitRange</i> z domyślnymi wartościami <b>250m CPU</b> oraz <b>256Mi pamięci RAM</b> dla pojedynczego kontenera. Zapewnia to stabilne działanie Pod-ów backendowych oraz zapobiega nadmiernemu rozdrobnieniu dostępnych zasobów.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/D5.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 24. LimitRange dla przestrzeni nazw backend (backend-limitrange.yaml)</i>
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/D6.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 25. Zastosowanie ResourceQuota oraz LimitRange w przestrzeni nazw backend</i>
</p>

<p align="justify">
Poprawność konfiguracji została zweryfikowana za pomocą poleceń <i>kubectl get quota</i> oraz <i>kubectl describe quota</i>, które potwierdziły poprawne naliczanie wykorzystania zasobów. Dodatkowo wykonano test przekroczenia limitu liczby Pod-ów w przestrzeni nazw <b>backend</b>. Próba utworzenia kolejnego Pod-a po osiągnięciu limitu została odrzucona z komunikatem o przekroczeniu quota, co jednoznacznie potwierdza poprawne działanie mechanizmu <i>ResourceQuota</i>.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/D7.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 26. Test przekroczenia limitu liczby Pod-ów w przestrzeni nazw backend</i>
</p>

<p align="justify">
Należy jeszcze raz podkreślić, że zarówno <i>ResourceQuota</i>, jak i <i>LimitRange</i> obowiązują wyłącznie dla nowo tworzonych Pod-ów. Istniejące Pod-y nie są modyfikowane ani usuwane automatycznie po wprowadzeniu ograniczeń. Takie zachowanie jest zgodne z dokumentacją Kubernetes i zostało uwzględnione podczas testów, a także w dalszej części sprawozdania.
</p>

---

### E. Konfiguracja Horizontal Pod Autoscaler (HPA) dla Deployment-u frontend
<p align="justify">
W ramach kolejnej części zadania utworzono obiekt <b>Horizontal Pod Autoscaler (HPA)</b> dla Deployment-u <b>frontend</b> w przestrzeni nazw <b>frontend</b>. Celem autoskalera jest dynamiczne dostosowywanie liczby replik Pod-ów w zależności od aktualnego obciążenia CPU, przy jednoczesnym maksymalnym wykorzystaniu zasobów dostępnych w tej przestrzeni nazw oraz bez możliwości przekroczenia ograniczeń narzuconych przez mechanizmy <i>ResourceQuota</i>.
</p>

<p align="justify">
Autoskalowanie zostało oparte o metrykę wykorzystania CPU wyrażoną jako procent wartości <i>requests.cpu</i>, zgodnie z algorytmem opisanym w dokumentacji Kubernetes. Liczba replik obliczana jest według wzoru:
</p>

```diff
desiredReplicas = currentReplicas × ( currentMetricValue / desiredMetricValue )
```

<p align="justify">
Oznacza to, że HPA skaluje Deployment proporcjonalnie do stosunku aktualnego wykorzystania CPU do wartości docelowej (<i>averageUtilization</i>). W konfiguracji autoskalera ustawiono próg <b>averageUtilization = 90%</b>, co oznacza, że HPA dąży do utrzymania średniego zużycia CPU na poziomie 90% wartości zadeklarowanych w <i>requests.cpu</i>. Takie ustawienie pozwala na bardziej agresywne, ale wciąż kontrolowane wykorzystanie dostępnych zasobów.
</p>

<p align="justify">
Zakres skalowania został ustawiony na <b>1–10 replik</b>, przy czym wartość maksymalna jest bezpośrednio zgodna z ograniczeniem <i>pods: 10</i> zdefiniowanym w obiekcie <i>ResourceQuota</i> dla przestrzeni nazw frontend. Dzięki temu autoskaler nie ma możliwości utworzenia większej liczby Pod-ów niż dopuszcza quota, nawet w przypadku wysokiego obciążenia. Zastosowanie obiektu <i>LimitRange</i>, który narzuca domyślną wartość <b>requests.cpu = 100m</b> dla pojedynczego Pod-a, zapewnia spójną podstawę do działania HPA, który podejmuje decyzje skalowania w oparciu o wartości <i>requests.cpu</i>, a nie <i>limits.cpu</i>. W efekcie 10 Pod-ów generuje łączne zapotrzebowanie na CPU na poziomie <b>1 CPU</b>, co gwarantuje maksymalne wykorzystanie dostępnych zasobów bez możliwości przekroczenia ograniczeń narzuconych przez <i>ResourceQuota</i>.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/E1.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 27. Włączenie dodatku metrics-server niezbędnego do działania HPA</i>
</p>

<p align="justify">
Po włączeniu <i>metrics-server</i> usunięto istniejące Pod-y frontend, aby nowe Pod-y zostały utworzone już z uwzględnieniem domyślnych wartości <i>requests</i> i <i>limits</i> narzuconych przez <i>LimitRange</i>. Pozwoliło to na prawidłowe zbieranie metryk CPU oraz ich wykorzystanie przez autoskaler.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/E2.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 28. Usunięcie, a następnie ponowne utworzenie Pod-ów frontend z uwzględnieniem LimitRange oraz widoczne zużycie quota</i>
</p>

<p align="justify">
Następnie utworzono obiekt <i>HorizontalPodAutoscaler</i> dla Deployment-u <i>frontend</i>.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/E3.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 29. Manifest Horizontal Pod Autoscaler dla Deployment-u frontend (frontend-hpa.yaml)</i>
</p>

<p align="justify">
Poprawność działania HPA została potwierdzona przy pomocy poleceń <i>kubectl get hpa</i> oraz <i>kubectl describe hpa</i>. Początkowo autoskaler utrzymywał liczbę replik zgodną z aktualnym stanem Deployment-u.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/E4.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 30. Utworzenie oraz początkowy stan obiektu HPA</i>
</p>

<p align="justify">
Po krótkim czasie działania autoskalera, przy niskim wykorzystaniu CPU, <b>HPA</b> automatycznie zmniejszył liczbę replik do wartości minimalnej, tj. <b>1</b>. Zachowanie to potwierdza poprawne działanie algorytmu autoskalowania oraz zgodność z ustawionym zakresem replik.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/E5.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 31. Automatyczne skalowanie Deployment-u frontend w dół przy niskim obciążeniu CPU</i>
</p>

<p align="justify">
Dzięki połączeniu mechanizmów <i>ResourceQuota</i>, <i>LimitRange</i> oraz <i>Horizontal Pod Autoscaler</i> uzyskano w pełni kontrolowany mechanizm autoskalowania. HPA może dynamicznie zwiększać lub zmniejszać liczbę Pod-ów w odpowiedzi na obciążenie CPU, maksymalnie wykorzystując dostępne zasoby, bez ryzyka przekroczenia limitów CPU, pamięci RAM ani maksymalnej liczby Pod-ów zdefiniowanych dla przestrzeni nazw frontend.
</p>

---

### G. Test obciążeniowy aplikacji frontend i weryfikacja działania autoskalera HPA
<p align="justify">
W ramach ostatniego elementu obowiązkowej części zadania przeprowadzono test obciążeniowy Deployment-u <i>frontend</i> w celu praktycznej weryfikacji poprawności działania mechanizmu <i>Horizontal Pod Autoscaler</i> oraz skuteczności wcześniej zdefiniowanych ograniczeń zasobów. Test został wykonany z wykorzystaniem narzędzia <b>fortio</b>, które generuje intensywny ruch HTTP do usługi <i>frontend</i> wystawionej jako <i>NodePort</i> i realne obciążenie CPU.
</p>

<p align="justify">
Obciążenie generowane było z poziomu osobnych Pod-ów uruchamianych w klastrze Kubernetes. Narzędzie <i>fortio</i> zostało skonfigurowane do pracy z dużą liczbą równoległych klientów (<b>-c 100</b>), bez limitu liczby zapytań na sekundę (<b>-qps 0</b>) oraz przez długi okres czasu (<b>-t 10m</b>). Uruchomionych zostało kilka konsol, w których generowane było różne obciążenie. Taka konfiguracja pozwoliła na osiągnięcie wysokiego i stabilnego zużycia CPU przez aplikację frontend. 
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/G1.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 32. Generowanie obciążenia CPU przy użyciu narzędzia fortio</i>
</p>

<p align="justify">
Ruch HTTP kierowany był do usługi <b>frontend</b> wystawionej jako <i>NodePort</i>. W trakcie testu monitorowano w czasie rzeczywistym stan obiektu HPA przy pomocy polecenia <i>kubectl get hpa -w</i>.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/G2.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 33. Dynamiczne skalowanie Deployment-u frontend w trakcie testu obciążeniowego</i>
</p>

<p align="justify">
Wraz ze wzrostem wykorzystania CPU, przekraczającym próg <b>averageUtilization = 90%</b>, autoskaler stopniowo zwiększał liczbę replik Deployment-u frontend. Liczba Pod-ów rosła proporcjonalnie do obciążenia, aż do osiągnięcia maksymalnej wartości <b>10 replik</b>, zgodnej z ograniczeniem <i>pods: 10</i> zdefiniowanym w obiekcie <i>ResourceQuota</i>.
</p>

<p align="justify">
W trakcie testu potwierdzono, że pomimo intensywnego obciążenia nie doszło do przekroczenia żadnego z limitów przestrzeni nazw <i>frontend</i>. Sumaryczne zapotrzebowanie na CPU osiągnęło dokładnie <b>1 CPU</b>, a liczba Pod-ów zatrzymała się na wartości maksymalnej dopuszczonej przez <i>quota</i>.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/G3.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 34. Maksymalne wykorzystanie quota CPU oraz liczby Pod-ów w przestrzeni nazw frontend</i>
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/G4.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 35. Wymagana ilość replik jest wyższa niż maksymalna dostępna</i>
</p>

<p align="justify">
Po zakończeniu generowania obciążenia <b>HPA</b> automatycznie rozpoczął proces skalowania w dół. Wraz ze spadkiem zużycia CPU liczba replik została stopniowo zredukowana aż do wartości minimalnej, zgodnej z konfiguracją autoskalera.
</p>

<p align="center">
  <img src="https://github.com/MarekP21/PFSwChO_Zadanie1/blob/main/screeny/G5.png" style="width: 80%; height: 80%" />
</p>
<p align="center">
  <i>Rys. 36. Automatyczne skalowanie Deployment-u frontend w dół po zakończeniu testu</i>
</p>

<p align="justify">
Przeprowadzony test jednoznacznie potwierdził poprawność konfiguracji <i>Horizontal Pod Autoscaler</i> oraz skuteczność zastosowanych mechanizmów <i>ResourceQuota</i> i <i>LimitRange</i>. Autoskaler dynamicznie reagował na zmiany obciążenia, maksymalnie wykorzystując dostępne zasoby CPU, jednocześnie nie przekraczając żadnych ograniczeń zdefiniowanych dla przestrzeni nazw frontend.
</p>

---

---

### Część nieobowiązkowa
<p align="justify"> 
  W ramach części nieobowiązkowej zadania przeanalizowano możliwość aktualizacji aplikacji <i>frontend</i> przy aktywnym <i>Horizontal Pod Autoscaler</i>. Odpowiedź brzmi <b>TAK</b>, możliwe jest dokonanie aktualizacji aplikacji frontend, gdy aplikacja jest pod kontrolą opracowanego autoskalera HPA. HPA kontroluje jedynie pole <i>replicas</i> obiektu Deployment i współpracuje z mechanizmem <i>rolling update</i> Deployment-u w taki sposób, że liczba Pod-ów w trakcie aktualizacji pozostaje zgodna z docelową wartością wyznaczoną przez HPA. W dokumentacji Kubernetes jednoznacznie wskazano, że autoskalowanie działa poprawnie w trakcie rolling update: <a href="https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#autoscaling-during-rolling-update">link do dokumentacji</a>.
</p> 

<p align="justify"> Podczas aktualizacji możliwe jest chwilowe istnienie Pod-ów z różnych wersji aplikacji, jednak łączna liczba Pod-ów pozostaje kontrolowana przez strategię <i>rollingUpdate</i> Deployment-u oraz przez HPA. Dzięki temu autoskalowanie działa poprawnie, a liczba Pod-ów nie przekracza ograniczeń narzuconych przez <i>ResourceQuota</i>. W celu zapewnienia, że podczas aktualizacji zawsze co najmniej dwa Pody realizują ruch aplikacji frontend oraz że nie zostaną przekroczone wcześniej zdefiniowane ograniczenia zasobów, zastosowano następujące parametry strategii RollingUpdate: </p>

```diff
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 0
```

<p align="justify"> Ponadto w HPA podniesiono wartość <i>minReplicas</i> z 1 do 3. Takie ustawienie gwarantuje, że:</p> 

<ul> 
  <li>nawet w najgorszym przypadku, gdy 1 Pod jest chwilowo niedostępny (<i>Unavailable</i>), co najmniej 2 Pody pozostają aktywne i obsługują ruch aplikacji,</li> 
  <li>nie zostaną przekroczone limity liczby Pod-ów ani zasoby CPU/RAM określone w <i>ResourceQuota</i>, ponieważ <i>maxSurge: 0</i> uniemożliwia tworzenie dodatkowych Pod-ów powyżej docelowej liczby replik,</li> 
  <li>autoskalowanie HPA pozostaje aktywne i spójne z procesem aktualizacji, jednocześnie zachowując bezpieczeństwo operacji.</li> 
</ul> 

<p align="justify"> Takie połączenie parametrów strategii RollingUpdate z dostosowaniem <i>minReplicas</i> w HPA zapewnia pełną kontrolę nad liczbą aktywnych Pod-ów, zgodność z limitami zasobów oraz bezpieczeństwo aktualizacji aplikacji frontend bez zakłócania działania autoskalera. </p>

---
