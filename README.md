# Websockets

Websoket server in PHP. Supporta anche connessioni SSL


## Modalità di utilizzo
##### Vi sono tre modi per utilizzare il server
1. Estendere la classe *WebSocketServer*
2. Implementare l'interfaccia *WebSocketConnection*
3. Passare le callbacks direttamente alla classe *WebSocketServer*

Sia con i primi due modi che con l'ultimo, si andranno a sovrascivere i seguenti metodi:
- *onConnect(Connection $conn, WebSocketServer $server)*: Questo metodo verrà invocato qunado un client si connette. Al momento della chiamata, gli verranno passati i dati sulla connessione e una istanza del server.
- *onDisconnect(Connection $conn, WebSocketServer $server)*: Questo metodo verrà invocato qunado un client si disconnette. Al momento della chiamata, gli verranno passati i dati  sulla connessione e una istanza del server.                                                       
- *onMessage(Connection $conn, $message, WebSocketServer $server)*: Questo metodo verrà invocato qunado un client invia un messaggio.  Al momento della chiamata, gli verranno passati i dati sulla connessione, il messaggio inviato e una istanza del server.
- *onTick(WebSocketServer $server) : bool*: Questo metodo verrà invocato periodicamente. Se il metodo restituisce 'false' il server smetterà di accettare richieste.
- *validateClient(Connection $conn, WebSocketServer $server) : bool*: Questo metodo verrà invocato quando un client si connette. Se il metodo restituisce 'false', verrà impedito al client di connettersi
                                                      
#### 1. Estendere la classe *WebSocketServer*
La classe che estende la class *WebSocketServer* deve sovrascrivere i metodi *onConnect()*, *onDisconnect()*, *onMessage()*, *onTick()* e  *validateClient()*.
###### ESEMPIO: 
``` php
# file Server.php

require "core/WebSocketServer.php";

class Server extends WebSocketServer
{
    public function onConnect(Connection $conn, WebSocketServer $server)// Metodo invocato quando un Client si connette
    {
        $server->send($conn->getSocket(), "Benvenuto!!"); // Invio un messaggio di benvenuto al Client appena connesso.
    }

    public function onDisconnect(Connection $conn, WebSocketServer $server)// Metodo invocato quando un Client si disconnette
    {
        $server->send($conn->getSocket(), "Arrivederci!!"); // Invio un messaggio al Client prima di liberare le risorse ad esso riservate
    }

    public function onMessage(Connection $conn, $message, WebSocketServer $server)// Metodo invocato quando un Client invia un messaggio
    {
        $server->send($conn->getSocket(), "HAI INVIATO:" . $message); // Restituisco il messaggio ricevuto
        $mes = "";
        foreach ($server->getClients() as $connection){
            $mes.= $connection->getID() . "\n";
        }
        $server->send($conn->getSocket(), "UTENTI CONNESSI:" . $mes);// Invio gli ID di tutti i Client connessi
    }

    public function onTick(WebSocketServer $server)// Metodo esseguito periodicamente. Se il metodo restituisce 'false' il server smetterà di accettare richieste
    {
        if(count($server->getClients()) > 100){ // se il numero di Client connessi è superiore a 100, il server si chiude
            return false;
        }
        return true;
    }

    public function validateClient(Connection $conn, WebSocketServer $server)
    {
        if ($conn->getHeaders()['host'] == "192.168.0.10"){
            return false; // non accetto connessione da un Client con ip = "192.168.0.10"
        }
        return true;
    }
}
```

```
# file start_server.php

require "Server.php";


try {
    $server = new Server();
}catch (Exception $e){
    exit("Il server nuo può essere avviato");
}

$server->run();
```
#### 2. Implementare l'interfaccia *WebSocketConnection*
Un oggetto della classe che implementa l'interfaccia *WebSocketConnection* deve essere passato come parametro nella classe *WebSocketServer*.
###### ESEMPIO: 
``` php
# file Client.php

require_once "core/WebSocketConnection.php";

class Client implements WebSocketConnection
{

        public function onConnect(Connection $conn, WebSocketServer $server){ // Metodo invocato quando un Client si connette
            $server->send($conn->getSocket(), "Benvenuto!!"); // Invio un messaggio di benvenuto al Client appena connesso.
        }
    
        public function onDisconnect(Connection $conn, WebSocketServer $server){// Metodo invocato quando un Client si disconnette
            $server->send($conn->getSocket(), "Arrivederci!!"); // Invio un messaggio al Client prima di liberare le risorse ad esso riservate
        }
    
        public function onMessage(Connection $conn, $message, WebSocketServer $server){// Metodo invocato quando un Client invia un messaggio
            $server->send($conn->getSocket(), "HAI INVIATO:" . $message); // Restituisco il messaggio ricevuto
            $mes = "";
            foreach ($server->getClients() as $connection){
                $mes.= $connection->getID() . "\n";
            }
            $server->send($conn->getSocket(), "UTENTI CONNESSI:" . $mes); // Invio gli ID di tutti i Client connessi
        }
    
        public function onTick(WebSocketServer $server){ // Metodo esseguito periodicamente. Se il metodo restituisce 'false' il server smetterà di accettare richieste
            if(count($server->getClients()) > 100){ // se il numero di Client connessi è superiore a 100, il server si chiude
                return false;
            }
            return true;
        }
    
        public function validateClient(Connection $conn, WebSocketServer $server){
            if ($conn->getHeaders()['host'] == "192.168.0.10"){
                return false; // non accetto connessione da un Client con ip = "192.168.0.10"
            }
            return true;
        }
}
```
```
# file start_server.php

require "core/WebSocketServer.php";
require "Client.php";

try {
    $server = new WebSocketServer(new Client());
}catch (Exception $e){
    exit("Il server nuo può essere avviato");
}

$server->run();
```
#### 3. Passare le callbacks direttamente alla classe *WebSocketServer*
###### ESEMPIO:
```
# file start_server.php

require "core/WebSocketServer.php";

try {
    $server = new WebSocketServer();
}catch (Exception $e){
    exit("Il server nuo può essere avviato");
}

$server->onConnectCallback(function (Connection $conn, WebSocketServer $server) {
    $server->send($conn->getSocket(), "Benvenuto!!"); // Invio un messaggio di benvenuto al Client appena connesso.
});// Metodo invocato quando un Client si connette

$server->onDisconnectCallback(function (Connection $conn, WebSocketServer $server) {
    $server->send($conn->getSocket(), "Arrivederci!!"); // Invio un messaggio al Client prima di liberare le risorse ad esso riservate
});// Metodo invocato quando un Client si disconnette

$server->onMessageCallback(function (Connection $conn, $message, WebSocketServer $server) {
    $server->send($conn->getSocket(), "HAI INVIATO:" . $message); // Restituisco il messaggio ricevuto
    $mes = "";
    foreach ($server->getClients() as $connection){
        $mes.= $connection->getID() . "\n";
    }
    $server->send($conn->getSocket(), "UTENTI CONNESSI:" . $mes); // Invio gli ID di tutti i Client connessi
}); // Metodo invocato quando un Client invia un messaggio

$server->onTickCallback(function (WebSocketServer $server){ // Metodo esseguito periodicamente. Se il metodo restituisce 'false' il server smetterà di accettare richieste
    if(count($server->getClients()) > 100){ // se il numero di Client connessi è superiore a 100, il server si chiude
        return false;
    }
    return true;
});

$server->validateClientCallback(function (Connection $conn, WebSocketServer $server){
    if ($conn->getHeaders()['host'] == "192.168.0.10"){
        return false; // non accetto connessione da un Client con ip = "192.168.0.10"
    }
    return true;
});

$server->run();
```
### La classe _Connection_
Questa classe mantine tutte le informazioni su una connessione attiva:


