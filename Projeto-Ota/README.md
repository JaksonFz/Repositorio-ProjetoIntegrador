# Projeto OTA


### Este codigo foi feito em c++ utilizando as bibliotecas: 

- Arduino.h Para executar os comandos do arduino;
  
- ESP8266WiFi.h Para utilizar funções relacionadas a conexão e autentição wifi; 

- ArduinoOTA.h Para executar funções OTA realizando as alterações remotas. 

### O fluxo do código acontece nas seguintes etapas: 

#### Setar as variaveis da rede escolhida(ssid e senha):

    const char* ssid = "Gustavojerba";
    const char* password = "wayofwade";

#### Fazer a conexão com a rede wifi: 

    WiFi.mode(WIFI_STA);
    WiFi.begin(ssid, password);

#### Conferir em loop se ele esta conectada, se o resultado da conexão for diferente de conectado ele imprime a mensagem de falha e tenta conectar novamente:

    while (WiFi.waitForConnectResult() != WL_CONNECTED) {
        Serial.println("Conexão falhou! Rebooting...");
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

 #### Por fim ira setar o loop do handler de comandos feitos para a placa:

    void loop() {
      ArduinoOTA.handle();
    }
