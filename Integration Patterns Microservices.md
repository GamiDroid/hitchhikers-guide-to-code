Microservices are small, and independent services that work together to form a larger application.

Event Driven Architecture, Message Queues, Point-to-Point channels and publish-subscibe enabled async communication between the services.

# Integration Patterns

## Shared Data

Transferring data via shared data. This can be done using files on disk. 

Dit kan betekenen dat service A een bestand in een bepaald formaat schrijft. Vervolgens kan service B het bestand oppakken.

Dit is een oudere versie van async communicatie tussen services.

## Request/ Response

Dit is de meest gebruikte communicatiestroom tussen services. Service A stuurt een request naar Service B. Service A verwacht vervolgens een response van Service B. Dit is dus een sync communicatiestroom.

"Design your interface if you would expect it to be a public interface". 

Request/ Response geeft tight coupling. Service A roept Service B aan. Service B vervolgens Service C. Als Service B vervolgens een error heeft wordt service C niet juist aangeroepen. Dit kan voor problemen zorgen. Het is daarom goed om microservice zo veel mogelijk te laten communiceren met async communicatiestromen.

Dit kan voor andere doeleinden goed worden gebruikt. Denk bijvoorbeeld aan de UI applicatie met de backend. Waar de gebruiker zo snel mogelijk een resultaat wilt krijgen.

## Publish/ Subscribe

	Publish/ subscribe is een scalable en flexible async communicatie model. 


