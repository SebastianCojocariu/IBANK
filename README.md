SEBASTIAN COJOCARIU 321CB UPB ACS

										                          IBANK

In vederea rezolvarii temei am folosit functia select() cu rolul de a mutiplexa
conexiuniile mai multor clienti(numar dinamic).In cele ce urmeaza vom descrie
cum am implementat fiecare comanda in parte:

Variabile importante:
   map-ul conturi in care am drept cheie numarul contului,iar drept element 
informatiile despre contul cu acel numar de cont salvate in TCont.
   map-ul users care are drept cheie indexul utilizatorului care face o cerere,
iar drept element are informatii referitoare la ultimul numar de card apelat in
incercarea de a face login,alaturi de numarul de incercari,salvate in TUtilizator.
   map-ul tranzactie care are drept cheie indexul utilizatorului care initiaza
o cerere de transfer,iar drept element informatiile despre nr_cont_sursa/destinatie,
alaturi de suma,toate acestea salvate in structura TTranzactie.
   map-ul users_conectati pe care il vom folosi pentru a tine evidenta utilizatorilor
care sunt conectati la server(in vederea trimiterii unui mesaj in cazul in care se
face "quit" de la server)


1)login: se verifica daca pt clientul i exista cheia i in users.Daca nu exista,se 
creeaza una,altfel daca exista insa nr_card primit nu e cel din map se reactualizeaza
numarul de card si nr de accesari aflate la cheia i.Altfel,inseamna ca e acelasi numar
de card,deci incrementam nr de accesari.Verificam apoi daca numarul de card primit
este unul valid,daca nu exista nimeni deja conectat pe el,iar apoi daca nu e blocat.
Daca nu se intampla niciun caz de mai sus,verificam daca pinul primit se potriveste
sau nu cu pinul contului.Daca nu e corect pinul,verificam daca nr_accesari a ajuns la 
3,caz in care blocam cardul si resetam nr_accesari la 0(si trimitem mesajul corespunzator
catre client),iar daca e corect setam clientul i ca activ pe contul respectiv si trimitem
mesajul "IBANK> Welcome....".In cadrul clientului,odata primit acest mesaj se va seta
variabila conectat la 1.

2)logout:reactualizam starea contului din care se face logout la 0 si trimitem catre client
mesajul "IBANK> Clientul a fost deconectat".In momentul in care primeste acest mesaj,
clientul reseteaza variabila conectat la 0.

3)listsold:trimite catre client mesajul cu soldul contului pe care e conectat.

4)transfer:serverul verifica mai intai ca numarul de card sa fie valid,respectiv suma
sa nu depaseasca soldul contului din care se initiaza cererea.Daca aceste 2 conditii
sunt satisfacute,se actualizeaza o TTranzactie si se introduce in map-ul tranzactie.
In acest moment,serverul asteapta sa primeasca y/n pentru a continua mai departe
cu procesul de transfer al banilor.In oricare din cazuri,la sfarsit cheia i si tranzactia
respectiva este eliminata din map ,iar un mesaj corespunzator este trimis clientului.

5)unlock:pentru aceasta comanda se va folosi un socket de UDP.Se verifica in prima faza
daca numarul de card exista,iar mai apoi daca contul respectiv chiar este blocat,cazuri
in care vor fi trimise mesaje corespunzatoare catre client.In caz contrar,un mesaj prin 
care i se solicita parola secreta va fi trimis catre client.Se verifica daca parola
primita e corecta,caz in care se deblocheaza contul,altfel se trimite
UNLOCK> -7 : Deblocare esuata".

6)quit:daca este initiat de catre server se va trimite mesajul "Serverul se va inchide.
Multumim pentru timpul vostru!IBANK va doreste o zi buna!" catre toti clientii conectati
la momentul respectiv dupa care se inchide serverul(si sesiunile clientilor).Daca
este primit de la client,in cazul in care e deja conectat la un cont se va elibera contul
respectiv,stergand totodata si cheia i a clientului din map-urile users,respectiv tranzactie.
Se elimina fd respectiv din multimea file-descriptorilor. 
