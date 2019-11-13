# Experiment: Avoid several threads cracking identical hashes simultaneously 

Ideen med eksperimentet er at undgå at flere tråde forsøger at knække det samme hash. På nuværende tidspunkt findes der 3 tråde, og det ønskes at hver tråd arbjeder på at knække unikke hashes. Priority queuen er baseret på et FIFO princip. Derfor skal der være X (vi talte om 5) antal requests mellem indentiske hashes i priority queues. På den måde regner vi med at det kan undgåes at trådene arbejder på ens hashes. 
Hvis et hash er blevet cracked lægges det i hashtablet, og en efterfølgdene identisk hash vil kunne slås op deri. Der vil således ikke opstå problemet med at trådende løser den samme hash. 

>Her er to eksempler på priority queues, hvor hash er blevet udskiftet med et int for lethedsskyld. 

I dette eksempel **(Eksempel 1)** ligger de to ens værdier tilpas langt fra hinanden. Det første 5 vil være blevet løst og lagt i hashtable inden turen kommer til det bagerste.
![](https://raw.githubusercontent.com/SelmaMeyer/Billeder_02159/master/queueOK.png)
I dette eksempel **(Eksempel 2)** ligger de to ens værdier for tæt på hinanden. Når alle de forgående request er løst, vil en tråd begynde at arbejde på det første 5, kort efter vil en anden tråd arbejde på det sidste 5. De arbejder altså samtidig på samme problem. Det er den situation vi ønsker at undgå. 
![](https://raw.githubusercontent.com/SelmaMeyer/Billeder_02159/master/QueueSKOD.png)
### Hvordan?

Tanken er, at der i hver request findes et sockArray, der kan indeholde op til 4 sockets. I tilfælde af at priority queuen vil indeholde det samme hash i træk, se eksempel 2, ønsker vi at ligge den nye requests socket ind i det sockArray, der findes i requesten med identisk hash. Denne ligger allerede i queuen og den nyeste requset vil ikke lægges i queue. Dog vil den givne sock modtage korrekt response via sockArray.

 **Eksempel 3:** Gammel metode. Alle requests lægges i queue. Hver request har sin egen sock, som int. Man risikerer at to tråde vil arbejde på den samme hash.  
![](https://raw.githubusercontent.com/SelmaMeyer/Billeder_02159/master/QueueSock.png)
 **Eksempel 4:** Ny metode. Socket fra den nyeste request lægges i sockArray, hos request, hvor hashen er identisk. Den nyeste request lægges IKKE i queue, da den identiske hashs requsest vil indeholde socket og derfor vil sende response til den korrekte sock.
![](https://raw.githubusercontent.com/SelmaMeyer/Billeder_02159/master/QueueSockArray.png)

### Kode og implementering

I filen `server.c` skal følgende metoder ændres: `send_response` og `read_request`. Når man sender en response tilbage til clienten skal der loopes igennem sockArray, og når requesten læses skal det nye sockArrray og index initialiseres. Desuden skal structen `requset` i `server.h` ændres, så den nu indeholder: 
- `int sockArray[4];` 
- `int sockArrayIndex;`   

Filen `queue.c` skal indeholde i hvert tilfælde to nye metoder. Begge metoderne skal kaldes i metoden `enQue(QUEUE *queue, REQ req)`, metoden `enQue` selv skal også modificeres. En ny metode skal tjekke hvorvidt den nye request der videregives til `enQue` allerede findes ibland de 5 nyeste request i alle 4 request arrays. Metoden kunne hedde `isInQue(QUEUE *queue, REQ newReq) ` og retunerer true, hvis den nye requests hash allerede eksisterer i queuen, ellers retuneres false. 

Hvis `isInQue` retunere `false` skal `enQue` bare gøre som den gør nu, og lægge den nye request i queue. 

Hvis `isInQue` retunere `true` skal den nye request IKKE lægges i queue. Dog skal man finde den request med identisk hash og tilføje den nye requests sock til sockArrayet for denne, som allerede findes i queue. Hvis der findes identisk hash i alle 4 af de 4 request arrays (`array, array2, array3, array4`), lægges den blot i den hvor dens prioritet matcher. 
Hvis fx identisk hash findes i 2 eller 3 af arraysne ved jeg ikke helt hvordan det skal håndteres bedst.
Hvis der kun findes en identisk hash lægges sock selvfølge i dennes tilhørende sockArray. Jeg er dog i tvivl om der kan opstå komplikationer i nogle tilfælde. Fx hvis den identiske hash har priotet 1, men den nye hash har prioritet 9. Så vi den komme i den langsomme queue. Derfor kan man overveje om der bør laves et tjek der kun indsætter sock i sockArray, hvis prioriteten er under eller lig med den identiske hashs prioritet. 

### Afklaringer
- Hvad skal man gøre hvis der findes identisk hashes i 2 eller 3 request arrays?
- Bør man tage højde for prioritet. Så man ikke kommer til at komme en prioritet 9 i en prioritet 1 queue?


// Selma 
