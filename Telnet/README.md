# Projeto Telnet:


### Este codigo foi feito em C++ utilizando as bibliotecas: 

- ESP8266WiFi.h Para utilizar funções relacionadas a conexão e autentição wifi; 

- ArduinoOTA.h Para executar funções OTA realizando as alterações remotas. 

### O fluxo do código acontece nas seguintes etapas: 

#### Setar as variaveis da rede escolhida(ssid e senha) e o numero de pino que será utilizado pelo led:

    const char* ssid = "Prof.Rafael";
    const char* password = "senhasenha";
    const int LED_PIN = 5; // D1

#### Fazer conexão com o servidor na porta 23 e declarar o client para receber e responder as atualizações da placa:

    WiFiServer server(23);
    WiFiClient client;

#### Fazer a conexão com a rede wifi: 

    WiFi.mode(WIFI_STA);
    WiFi.begin(ssid, password);

#### Conferir em loop se ela esta conectada, se o resultado da conexão for diferente de conectado ele imprime a mensagem de falha e tentara conectar novamente:

    while (WiFi.waitForConnectResult() != WL_CONNECTED) {
        Serial.println("Conexão falhou, tentando novamente...");
        WiFi.begin(ssid, password);
        delay(5000);
    }

#### Cria a estrutura de handlers do OTA com seus respectivos retornos para cada situação:

- Inicio:

        ArduinoOTA.onStart([]() {
        Serial.println("Start");
        });

- Fim:

        ArduinoOTA.onEnd([]() {
        Serial.println("\nEnd");
        });

- Em progresso:

        ArduinoOTA.onProgress([](unsigned int progress, unsigned int total) {
        Serial.printf("Progress: %u%%\r", (progress / (total / 100)));
        });

- Possiveis erros:

        ArduinoOTA.onError([](ota_error_t error) {
            Serial.printf("Error[%u]: ", error);
            if (error == OTA_AUTH_ERROR) Serial.println("Auth Failed");
            else if (error == OTA_BEGIN_ERROR) Serial.println("Begin Failed");
            else if (error == OTA_CONNECT_ERROR) Serial.println("Connect Failed");
            else if (error == OTA_RECEIVE_ERROR) Serial.println("Receive Failed");
            else if (error == OTA_END_ERROR) Serial.println("End Failed");
        });

#### Inicia a conexão OTA e retorna o IP da placa: 

    ArduinoOTA.begin();
    Serial.println("Pronto");
    Serial.print("IP: ");
    Serial.println(WiFi.localIP());

#### Inicia o servidor Telnet e configura para que as informações que serão enviadas não esperem para ser divididas em pacotes menores antes:

    server.begin();
    server.setNoDelay(true);
    Serial.println("Servidor Telnet iniciado...");

#### Monta o loop do handler de comandos feitos para a placa. Além disso gerencia os clientes nessas condições: se há um cliente e se ele está ativo, encerra clientes inativos e cria novas instancias de cliente. Se a aplicação tiver um cliente será retornada uma interface com os comandos disponiveis:

    void loop() {

    ArduinoOTA.handle();

    if (server.hasClient()) {

    if (!client || !client.connected()) {

        if (client) client.stop();

        client = server.available();

        Serial.println("Cliente conectado via Telnet.");

        client.println("Digite '1' para ligar o LED e '0' para desligar o LED.");

    } else {

        server.available().stop();

    }

    }

#### Por fim configura os comandos a serem executados pelo cliente, nesse caso manipula o estado do led/lampada após mais uma verificação de cliente ativo ou inativo e declara a escuta dos clientes:

- Verificação e escuta de comandos:

        if (client && client.connected() && client.available())
          {

        char command = client.read();

- Condicional Ligado(1):  

        if (command == '1')
        {
    
          digitalWrite(LED_PIN, LOW);
    
          client.println("LED ligado.");
    
          Serial.println("LED ligado.");
        }

- Condicional Desligado(0):
    
        else if (command == '0')
        {
    
          digitalWrite(LED_PIN, HIGH);
    
          client.println("LED desligado.");
    
          Serial.println("LED desligado.");
        }

- Condicional para opção invalida:

        else
        {
    
          client.println("Comando inválido. Digite '1' para ligar o LED e '0' para desligar o LED.");
        }
