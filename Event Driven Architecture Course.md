# "Wat is betekend een "Event" en "Event Driven" volgens mij?" 
Ik zie een event als een trigger. Iets gebeurt in een periode in tijd en dit wordt vervolgens gepublisht. Een event of trigger kan een andere proces starten. Dit zorgt dat het event driven wordt.

# Coupling
"Coupling" is de schaal van waarvan een module afhankelijk is van een andere module. Lage "Coupling" betekend dat de module is onafhankelijk. Hoge "Coupling" is veel afhankelijkheid. Een wijziging in de ene module zorgt waarschijnlijk voor een wijziging in de andere module.

# Cohesion
"Cohesion" is de graad waarin een element in de module samen werkt om één gerichte taak uit te voeren. Hoge "Cohesion" betekend dat elementen veel relatie hebben met hetzelfde doel. Lage "Cohesion" betekend dat ze meerdere doelen hebben. 

In software ontwikkeling is een doel om "Low coupling and High Cohesion".

# Types of messages
Er zijn over het algemeen drie verschillende types berichten: 
- Commands
- Events
- Queries

"Commands" is een systeem die zegt dat een andere systeem iets moet doen. Bijv. 'StartOrder', 'BookPallet'.

"Events" verteld tegen een ander systeem dat iets is gebeurt. Bijv. 'OrderStarted', 'PalletBooked'.

"Queries" is het opvragen van gegevens van een ander systeem. Bijv. Geef mij de "Order die zijn gestart".

# Types of events
De type events die veel worden gebruikt is "Notification event". ook wel "Thin event". Een systeem laat aan een ander systeem weten dat iets is gebeurt.

In de afbeelding hieronder kan je zien dan de "Order Processing Service" een event "Order Confirmed" event aanmaakt. De "Kitchen Service" wilt luistert naar het event om het gericht te gaan maken. In het Event is alleen het OrderId bekent. De "Kitchen Service" moet nogsteeds de gegevens ophalen uit de "Order Processing Service". Wat de "Coupling" niet verminderd.
![[notification-event.png]]

Daarnaast weet een systeem die een event published niet hoeveel consumers er zijn. Als alle consumers nu een order moeten ophalen nadat een event is gepublished dan kan de "Order Processing Service" misschien wel een hele hoge load krijgen.

Een ander type event is een "Event Carried State Event" ook wel "Thick Event". De event levert direct meer data aan de consumer. Dus de noodzaak om een callback te doen wordt hiermee verkleind.

```csharp
// Notification Event (Thin)
public record OrderConfirmedEvent(int OrderId);

// Event Carrier State Event (Thick)
public record OrderConfirmedEvent(
	int OrderId,
	string OrderNumber,
	double OrderValue,
	DeliveryAddress DeliveryAddress,
	List<Item> Items
);
```

Je verhoogt hiermee wel "Coupling" aangezien de data format van je event tussen de systemen overeen moeten blijven.

Wat moet je nou gebruiken? 'Thin' of 'Thick' events? 

# Event Storming
Als je een systeem wilt gaan maken, hoe weet je nou welke events er zijn. Hierbij kan "Event storming" helpen. "Event storming" is net als brainstorming een lijst van events bedenken die nodig zijn voor het proces. Dit kan fundamenteel helpen bij het aanmaken van events. DIT IS GEEN TECHNISCHE DISCUSSIE!

# Message Channels
Er zijn twee types "message channels": 
- "Point-to-Point" (P2P)
- "Publish/Subscribe" (Pub/Sub)

P2P is handig als de ene module directe communicatie nodig heeft met een andere module. Het kan ook meerdere instanties van dezelfde module zijn voor load balancing. Maar geen andere modules. Hiervoor moet dan een andere P2P channel voor gemaakt worden.

Pub/Sub is als een module naar mogelijk meerdere modules. Een module subscibed voor berichten van een andere module.

## Channel types
P2P wordt vaak met een "Queue" geregeld. Een BELANGRIJK concept van een queue is, dat deze persistent (durable) is. Als module A een bericht aanmaakt en module B is offline. Dat het bericht nog wel in de queue beschikbaar is.

> [!NOTE]
> Veel queuing mechanisme garanderen niet dat de ordering hetzelfde is als dat het in de queue is gekomen. SQS geeft bijvoorbeeld wel de mogelijkheid om FIFO in te schakelen. Bedenk of het ordering belangrijk is voor het proces.

Bus/ Topic is vaak niet persistent. Als een module subscibes naar een topic, deze berichten worden verwerkt. Komt er vervolgens een module bij, pakt deze de berichten vanaf dat moment op.

"Stream" is een combinatie van de twee. De berichten worden in een queue gezet. Deze berichten worden door de consumer verwerkt. Valt de module toch weg en wordt deze opnieuw opgestart wordt vanaf het moment waar de module is gebleven het weer verder op. 

# Producer
## "Wat denk ik dat een goede producer moet doen?"
Een producer moet duidelijk aangeven wat er proces is gebeurt bij een specifiek event. "OrderCompleted" geeft bijvoorbeeld heel duidelijk aan dat het om een order gaat en dat deze klaar is. Het is ook belangrijk om genoeg data mee te geven zodat de consumer weet over welk entiteit het precies gaat.

# Public / Private events
Als je een events aan het defineren bent is het goed op na te denken over of de event "public" of "private" is. Als een extern systeem bij de topic kan is het een public event. Wanneer je zelf de andere module in beheer hebt is het event "private". Is een "private" event mag meer informatie beschikbaar zijn aangezien deze precies kan doen wat je nodig hebt.

# UI communication in async world
Hoe laat je de gebruiker nou weten dat "iets" is gebeurt. Hiervoor zijn twee mechanismes voor: Polling of Web sockets.

## Polling
Er wordt continue een request naar de server gedaan om te kijken of er een update is.

## Web socket
De client krijgt een update gepusht vanuit de service.