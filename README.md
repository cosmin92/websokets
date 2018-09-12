# Websockets

Server in PHP basato su Websocket.<br>
Questo pacchetto implementa il protocollo Websocket secondo le specifiche [IETF RFC6455 ](https://tools.ietf.org/html/rfc6455)


## Modalità di utilizzo
###### Vi sono tre modi per utilizzare il server
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
```php
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

```php
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
Un oggetto della classe che implementa l'interfaccia *WebSocketConnection* deve essere passato come parametro nel costruttore della classe *WebSocketServer*.
###### ESEMPIO: 
```php
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
```php
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
```php
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
La classe _Connection_  mantiene tutte le informazioni su una connessione attiva.
###### API:
  - _&getSocket(): resource_ . Restituisce la socket relativa alla conessione corrente. 
  - _getHeaders(): array_ . Restituisce un array con le informazioni sul client connesso.
  - _getResource(): string_ . Restituisce una stringa; il path della risorsa richiesta dal Client
  - _getID(): string_ . Restituisce una stringa; ID = md5(uniqueid() + mt_rand(1, 12345)).
  - _getTimeStamp(): int_ . Restituisce sottoforma di intero il momento in cui il Client si è connesso.

### La classe _WebSocketServer_
La classe  _WebSocketServer_, sottoclasse della classe _WebSocket_, implementa il protocollo Websocket. Crea inoltre, una socket sulla quale accetta le connessioni che ripettano il protocollo Websocket.
In aggiunta è possibile definire _host_, _port_, _scheme_, etc. nel file config.ini.
###### API:
 - _construct(WebSocketConnection $connection = null, OpenSSL $ssl = null)_ . Construttore: accetta come paremetri un oggetto di una classe che implementa l'interfaccia WebSocketConnection e un oggetto della classe OpenSSl per instaure connessioni sicure sul protocollo TLS. 
 - _run(): void_ . Avvia il server.
 - _getClients(): array_. Retituisce un array con gli oggetti della classe Connection(tutte le connessioni attive).  
 - _getServer(): resource_. Restituisce la socket su qui il server è in ascolto.
 - _log($message)_ . Scrive il messaggio, passato come parametro, nel file 'server.log'.   
 - _validateClientCallback(callable $callback)_. Imposta una callback che verrà invocata quando un client si connette. Quando viene invocata la callback, gli vengono passati i seguaenti parametri: 1) un'stansa della classe Connection. 2) un'istanza del server corrente. La callback deve ritornare 'true' se al client viene dato il permesso di connettersi. Altrimenti 'false'.
 - _onConnectCallback(callable $callback)_. Imposta una callback che verrà invocata quando un client si connette. Quando viene invocata la callback, gli vengono passati i seguaenti parametri: 1) un'stansa della classe Connection 2) un'istanza del server corrente.
 - _onDisconnectCallback(callable $callback)_. Imposta una callback che verrà invocata quando un client si disconnette. Quando viene invocata la callback, gli vengono passati i seguaenti parametri: 1) un'stansa della classe Connection. 2) un'istanza del server corrente.
 - _onMessageCallback(callable $callback)_. Imposta una callback che verrà invocata quando un client invia un messaggio. Quando viene invocata la callback, gli vengono passati i seguaenti parametri: 1) un'stansa della classe Connection. 2) il messaggio ricevuto. 3) un'istanza del server corrente.
 - _onTickCallback(callable $callback)_ . Imposta una callback che verrà invocata periodicamente. Quando viene invocata la callback, gli vengono passati i seguaenti parametri: 1) un'istanza del server corrente. Se il metodo restituisce 'false' il server smetterà di accettare richieste.

 - **_static function wakeUp($script_path)_**. Riceve come parametro il path dello script php che lancia il server. Se il server e offline essegue lo script.


