 /*
  autor: bruhosabt 09/11/2023
  PIBID - FÍSICA 
  QUEDA LIVRE
  versão 0.2.1 -> odificado e concluído: 14/11/2023 ~ bhst
  Modificado : 29/02/24 ~ bhst

  Código desenvolvido para o projeto de Queda Livre desenvolvido no âmbito do Programa Institucional de Bolsas de iniciação à Docência.

  -------------x-x-x-x-x-------------
              FÍSICA 2023
  -----------------------------------
Versão 0.2.2 de 2024
conclúida em: 27/03/2024
Autor: bruno da silva santos
*/

#include <Servo.h>
#include <string.h>
#include <Arduino.h>
#include <U8g2lib.h>

//pinos digitais
const int LED_1 = 4 ;
const int LED_2 = 5;
const int LED_3 = 6;
const int LED_4 = 7;
//pinos analógicos
const int LDR_1 = A2;
const int LDR_2 = A3;
const int LDR_3 = A4;
const int LDR_4 = A5;

const int SERVO = 3; // PWD, pino digital ~

//limiar de início de queda e fim de queda, deixou de ser constante, pois agora pode ser modificado no teste
const int max_ldr = 50;
int min_ldr = 610;
//pinos digitais para os botões
const int btn_esquerdo = 8;
const int btn_centro = 2 ;// a mudança deste pino não é opcional, deve permanecer o pino 2, pois ele é necessário para a attachInterrupt
const int btn_direito = 9;
//tamanho da tela
const int SCREEN_WIDTH = 128;
const int SCREEN_HEIGHT = 64;
const int OLED_RESET = -1;

//estado dos botões
int estado_btn_esquerdo = HIGH;
int estado_btn_centro = HIGH;
int estado_btn_direito = HIGH;

int ultimoEstadoEsquerdo = HIGH;
int ultimoEstadoCentro = HIGH;
int ultimoEstadoDireito = HIGH;
//tempo de devounce (ms), pode ser alterado
unsigned long debounceDelay = 10;

//tempo da última mudança

unsigned long ultimoDebounceTime_esquerdo = 0;
unsigned long ultimoDebounceTime_centro = 0;
unsigned long ultimoDebounceTime_direito = 0;
//variáveis para menus
const char menu_principal_1 [] PROGMEM = "INICIAR";
const char menu_principal_2 [] PROGMEM = "DADOS";
const char menu_principal_3 [] PROGMEM = "ALTURA";
const char menu_principal_4 [] PROGMEM = "TESTE";
const char* const menu_principal [] PROGMEM ={menu_principal_1, menu_principal_2, menu_principal_3, menu_principal_4};

//const char * menu_principal [] ={"INICIAR","DADOS","ALTURAS"};

const char menu_dados_1 [] PROGMEM = "GRAVIDADE";
const char menu_dados_2 [] PROGMEM = "VELOCIDADE";
const char menu_dados_3 [] PROGMEM = "ERRO";
const char menu_dados_4 [] PROGMEM = "VOLTAR";
const char* const menu_dados[] PROGMEM ={menu_dados_1,menu_dados_2, menu_dados_3, menu_dados_4};
//const char * menu_dados [] = {"GRAVIDADE","VELOCIDADE","ERRO","VOLTAR"};

const char menu_alturas_1 [] PROGMEM = "0.3m";
const char menu_alturas_2 [] PROGMEM = "0.6m";
const char menu_alturas_3 [] PROGMEM = "0.9m";
const char menu_alturas_4 [] PROGMEM = "VOLTAR";
const char* const menu_alturas[] PROGMEM = {menu_alturas_1, menu_alturas_2, menu_alturas_3,menu_alturas_4};
//const char * menu_alturas [] = {"0.3m","0.6m","0.9m","VOLTAR"};

//adição
const char menu_teste_1 [] PROGMEM = "0m";
const char menu_teste_2 [] PROGMEM = "0.3m";
const char menu_teste_3 [] PROGMEM = "0.6m";
const char menu_teste_4 [] PROGMEM = "0.9m";
const char menu_teste_5 [] PROGMEM = "VOLTAR";
const char* const menu_teste[] PROGMEM = {menu_teste_1, menu_teste_2, menu_teste_3, menu_teste_4, menu_teste_5};
//const char * menu_teste [] = {"0m","0.3m","0.6","0.9","VOLTAR"};
bool dentro_m_teste = false;
int contador_m_teste = 0;
int aux_teste_in = 0;

//fim adição
bool dentro_m_principal = true; // verdade inicialmente, para mostra ele primeiro
bool dentro_m_dados = false;
bool dentro_m_alturas = false;

int contador_m_principal, contador_m_dados, contador_m_alturas = 0;

const float alturas [] = {0.30, 0.60, 0.90};
int index_altura = -1;
bool dentro_start = false; // para controle do reset
bool check = false;

unsigned long tempo1, tempo2;
float deltaTempo, gravidade, velocidade, erro = 0;

const float g_referencial = 9.79; // para cálculo do erro percentual
// para colocar os valores convertidos para string
char str_gravidade[15]; 
char str_velocidade[15];
char str_erro[15];
//para controle de exibição dos menus
int anterior_altura_in_contador = 0;
int anterior_principal_in_contador = 0;
int anterior_dados_in_contador = 0;
int aux_dados_in = 0;
//adição
int anterior_teste_in_contador = 0;
//fim adição

volatile bool cancelar_captura = false;//para controle do reset

int aux_altura_in = 0;
U8G2_ST7920_128X64_F_SW_SPI display(U8G2_R0, 13, 11, 10, U8X8_PIN_NONE);
Servo garra; //instância objeto do servo

void setup()
{
  pinMode(LED_1, OUTPUT);
  pinMode(LED_2, OUTPUT);
  pinMode(LED_3, OUTPUT);
  pinMode(LED_4, OUTPUT);

  pinMode(btn_esquerdo, INPUT_PULLUP);
  pinMode(btn_centro, INPUT_PULLUP);
  pinMode(btn_direito, INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(2), reset, FALLING);

  garra.attach(SERVO);

  display.begin();
  delay(1000);//espera 2 segundos
  ///print_logo();
  delay(1000);
  print_menu_principal();
  garra.write(2);

}

void loop()
{
  int leitura_esquerdo = digitalRead(btn_esquerdo);
  int leitura_centro = digitalRead(btn_centro);
  int leitura_direito = digitalRead(btn_direito);
 //*==================*#*==================*
 //ultimoEstadoEsquerdo
  if(leitura_esquerdo != ultimoEstadoEsquerdo)//navegação decrescente
  {
    ultimoDebounceTime_esquerdo = millis();
  }
  if((millis() - ultimoDebounceTime_esquerdo) > debounceDelay)
  {
    if(leitura_esquerdo != estado_btn_esquerdo)
    {
      estado_btn_esquerdo = leitura_esquerdo;
      if(estado_btn_esquerdo == LOW)
      {
        if(dentro_m_principal)//navega dentro do menu principal, decrescente
        {
          if(contador_m_principal > 0)
          {
            contador_m_principal -=1;

          }else{

            contador_m_principal = 3;
          }
          if(anterior_principal_in_contador != contador_m_principal)
          {
            contador_m_dados = 0;
            contador_m_alturas = 0;
            contador_m_teste = 0;
            anterior_principal_in_contador = contador_m_principal;
            print_menu_principal();
          }
        }
        if(dentro_m_dados)//navega dentro do menu dados, decrescente
        {
          if(contador_m_dados > 0)
          {
            contador_m_dados -= 1;

          }else{

            contador_m_dados = 3;
          }
          if(anterior_dados_in_contador != contador_m_dados)
          {
            contador_m_alturas = 0;
            contador_m_teste = 0;
            contador_m_principal = 0;
            anterior_dados_in_contador = contador_m_dados;
            print_menu_dados();
          }
        }
        if(dentro_m_alturas)//navega dentro do menu alturas, decrescente
        {
          if(contador_m_alturas > 0)
          {
            contador_m_alturas -= 1;
          }else{
            contador_m_alturas = 3;
          }
          if(anterior_altura_in_contador != contador_m_alturas)
          {
            contador_m_dados = 0;
            contador_m_teste = 0;
            contador_m_principal = 0;
            anterior_altura_in_contador = contador_m_alturas;
            print_menu_alturas();
          }
        }
          //adição
        if(dentro_m_teste)
        {
          if(contador_m_teste > 0)
          {
            contador_m_teste -= 1;
          }else{
            contador_m_teste = 4;
          }
          if(anterior_teste_in_contador !=contador_m_teste)
          {
            contador_m_alturas = 0;
            contador_m_dados = 0;
            contador_m_principal = 0;
            anterior_teste_in_contador = contador_m_teste;
            print_menu_teste();
          }
        }
      
      }//fim terceiro
     
    }//fim segundo
  }//primeiro if
  ultimoEstadoEsquerdo = leitura_esquerdo;
 
  //*==================*#*==================*
  if(leitura_direito != ultimoEstadoDireito)//navegação crescente
  {
    ultimoDebounceTime_direito = millis();
  }
  if((millis() - ultimoDebounceTime_direito) > debounceDelay)
  {
    if(leitura_direito != estado_btn_direito)
    {
      estado_btn_direito = leitura_direito;
      if(estado_btn_direito == LOW)
      {
        if(dentro_m_principal)//navega dentro do menu principal, crescente
        {
          if(contador_m_principal < 3)
          {
            contador_m_principal += 1;
          }else{
            contador_m_principal = 0;
          }
          if(anterior_principal_in_contador != contador_m_principal)
          {
            contador_m_dados = 0;
            contador_m_alturas = 0;
            contador_m_teste = 0;
            anterior_principal_in_contador  = contador_m_principal;
            print_menu_principal();
          }

        }
        if(dentro_m_dados)// navega dentro do menu dados, crescente
        {
          if(contador_m_dados < 3)
          {
            contador_m_dados += 1;
          }else{
            contador_m_dados = 0;
          }
          if(anterior_dados_in_contador != contador_m_dados)
          {
            contador_m_alturas = 0;
            contador_m_principal = 0;
            contador_m_teste = 0;
            anterior_dados_in_contador = contador_m_dados;
            print_menu_dados();
          }
        }
        if(dentro_m_alturas)//navega dentro do menu alturas, crescente
        {
          if(contador_m_alturas < 3)
          {
            contador_m_alturas += 1;
          }else{
            contador_m_alturas = 0;
          }
          if(anterior_altura_in_contador != contador_m_alturas)
          {
            contador_m_dados = 0;
            contador_m_principal = 0;
            contador_m_teste = 0;
            anterior_altura_in_contador = contador_m_alturas;
            print_menu_alturas();
          }
        }
        //adição
        if(dentro_m_teste)
        {
          if(contador_m_teste < 4)
          {
            contador_m_teste += 1;
          }else{
            contador_m_teste = 0;
          }
          if(anterior_teste_in_contador !=contador_m_teste)
          {
            contador_m_alturas = 0;
            contador_m_dados = 0;
            contador_m_principal = 0;
            anterior_teste_in_contador = contador_m_teste;
            print_menu_teste();
          }
        }
      }
    }
  }
   
  ultimoEstadoDireito = leitura_direito;
  //*==================*#*==================*
  if(leitura_centro != ultimoEstadoCentro)
  {
    ultimoDebounceTime_centro = millis();
  }

  if((millis()) - ultimoDebounceTime_centro > debounceDelay)
  {
    if(leitura_centro != estado_btn_centro)
    {
      estado_btn_centro = leitura_centro;
      if(estado_btn_centro == LOW)//enter foi pressionado, mas onde? Segue..
      {
        if(dentro_m_principal)//se o enter foi pressionado dentro do menu principal, faz
        {
          switch (contador_m_principal)//verifica onde o contador do menu principal está
          {
            case 0: //caso esteja em 0, então o enter foi pressionado para iniciar a captura dos dados
              dentro_m_principal = false;
              dentro_m_alturas = false;
              dentro_m_dados = false;
              dentro_m_teste = false;
              iniciar();
            break;
            case  1:
              dentro_m_principal = false;
              dentro_m_alturas = false;
              dentro_m_teste = false;
              dentro_m_dados = true;
              print_menu_dados(); // caso seja 1, então significa que o usuário quer ver os dados
            break;
            case 2: // caso seja 2, então significa que o usuário quer selecionar a altura
              dentro_m_principal = false;
              dentro_m_alturas = true;
              dentro_m_teste = false;
              dentro_m_dados = false;
              print_menu_alturas();
            break;
            case 3:
              dentro_m_principal = false;
              dentro_m_alturas = false;
              dentro_m_dados = false;
              dentro_m_teste = true;
              print_menu_teste();
            break;
            default:
            break;
          }
        }
        if(dentro_m_dados)//enter pressionado dentro do menu dados
        {
          switch (contador_m_dados)
          {
            case 0: //o usuário quer ver a gravidade
              if(aux_dados_in > 0)
              {
                print_gravidade();
                aux_dados_in = 0;
              }else{
                aux_dados_in += 1;
              }
            break;
            case 1:
              print_velocidade(); //o usuário quer ver a velocidade
            break;
            case 2:
              print_erro(); // ... quer ver o erro percentual
            break;
            case 3:
              dentro_m_principal = true;
              dentro_m_alturas = false;
              dentro_m_dados = false;
              dentro_m_teste = false;
              voltar(); //... quer voltar
            break;
            default:
            break;
          }
        }
        if(dentro_m_alturas) // enter pressionado dentro do menu alturas
        {
          switch(contador_m_alturas)
          {
            case 0:/* se em qualquer caso terei de pegar a altura selcionada, então por que não pular as etapas.*/
            case 1:
            case 2:
              if(aux_altura_in > 0)
              {
                index_altura = contador_m_alturas;
                // print_altura_selecionada();
                dentro_m_alturas = false;
                dentro_m_dados = false;
                dentro_m_principal = true;
                aux_altura_in  = 0;
                voltar();
              }
              else{
                aux_altura_in ++;
              }
            break;
            case 3:
              dentro_m_principal =true;
              dentro_m_alturas = false;
              dentro_m_dados = false;
              dentro_m_teste = false;
              voltar();
            break;
            default:
            break;
          }
        }
        if(dentro_m_teste)
        {
          switch(contador_m_teste)
          {
            case 0:
              if(aux_teste_in >0)
              {
                testeSensores(LED_1, LDR_1);
                aux_teste_in = 0;
              }else
              {
                aux_teste_in +=1;
              }
            break;
            case 1:
              testeSensores(LED_2, LDR_2);
            break;
            case 2:
              testeSensores(LED_3, LDR_3);
            break;
            case 3:
              testeSensores(LED_4, LDR_4);
            break;
            case 4:
              dentro_m_principal = true;
              dentro_m_teste = false;
              dentro_m_dados = false;
              dentro_m_alturas = false;
              voltar();
            break;
            default:
            break;
          }
        }
      }// fim 2 if
    }
  }
  ultimoEstadoCentro = leitura_centro;
}

void print_menu_principal()
{
  display.clearBuffer(); // Limpa o buffer do display
  display.setFont(u8g2_font_ncenB08_tr); // Define a fonte

  // Título do Menu
  display.drawStr(0, 10, "MENU PRINCIPAL");

  // Quantidade de opções no menu
  int numeroDeOpcoes = sizeof(menu_principal) / sizeof(char*);

  // Altura inicial para a primeira opção
  int alturaInicial = 20;
  char buffer[30]; // Buffer para armazenar strings temporárias

  for (int i = 0; i < numeroDeOpcoes; i++) {
    int xOffset = 20; // Posição x padrão para o texto das opções

    // Carrega a string do menu principal da memória flash para o buffer
    strcpy_P(buffer, (char*)pgm_read_word(&(menu_principal[i])));

    if (i == contador_m_principal)
    {
      // Alterna para a fonte de ícones
      display.setFont(u8g2_font_open_iconic_arrow_1x_t);
      // Desenha o ícone de seta
      display.drawGlyph(10, alturaInicial + i * 10, 0x004E);
      // Volta para a fonte original
      display.setFont(u8g2_font_ncenB08_tr);
      xOffset += 18; // Desloca o texto da opção selecionada para a direita
    }

    // Desenha o texto da opção
    display.drawStr(xOffset, alturaInicial + i * 10, buffer);
  }

    display.sendBuffer(); // Envia o buffer para o display
}

void print_menu_dados()
{
  display.clearBuffer(); // Limpa o buffer do display
  display.setFont(u8g2_font_ncenB08_tr); // Define a fonte

  // Título do Menu
  display.drawStr(20, 10, "MENU DADOS");

  // Quantidade de opções no menu
  int numeroDeOpcoes = sizeof(menu_dados) / sizeof(char*);

  // Altura inicial para a primeira opção
  int alturaInicial = 20;
  char buffer[30]; // Buffer para armazenar strings temporárias

  for (int i = 0; i < numeroDeOpcoes; i++)
  {
    int xOffset = 20; // Posição x padrão para o texto das opções

    // Carrega a string do menu de dados da memória flash para o buffer
    strcpy_P(buffer, (char*)pgm_read_word(&(menu_dados[i])));

    if (i == contador_m_dados)
    {
      // Alterna para a fonte de ícones
      display.setFont(u8g2_font_open_iconic_arrow_1x_t);
      // Desenha o ícone de seta
      display.drawGlyph(10, alturaInicial + i * 10, 0x004E);
      // Volta para a fonte original
      display.setFont(u8g2_font_ncenB08_tr);
      xOffset += 10; // Desloca o texto da opção selecionada para a direita
    }

    // Desenha o texto da opção
    display.drawStr(xOffset, alturaInicial + i * 10, buffer);
  }

  display.sendBuffer(); // Envia o buffer para o display
}

void print_menu_alturas()
{
  display.clearBuffer();
  display.setFont(u8g2_font_ncenB08_tr);

  // Título do Menu
  display.drawStr(20, 10, "MENU ALTURAS");

  // Quantidade de opções no menu
  int numeroDeOpcoes = sizeof(menu_alturas) / sizeof(char*);

  // Altura inicial para a primeira opção
  int alturaInicial = 20;
  char buffer[30];
  for (int i = 0; i < numeroDeOpcoes; i++)
  {
    int xOffset = 20; // Posição x padrão para o texto das opções
    strcpy_P(buffer, (char*)pgm_read_word(&(menu_alturas[i])));
    if (i == contador_m_alturas) {
      // Alterna para a fonte de ícones
      display.setFont(u8g2_font_open_iconic_arrow_1x_t);
      // Desenha o ícone de seta
      display.drawGlyph(20, alturaInicial + i*10, 0x0004e); // O código do ícone pode variar
      // Volta para a fonte original
      display.setFont(u8g2_font_ncenB08_tr);
      xOffset += 10; // Desloca o texto da opção selecionada para a direita
    }

    // Desenha o texto da opção
    display.drawStr(xOffset, alturaInicial + i*10, buffer);
  }

  display.sendBuffer();
}
void print_menu_teste()
{
  display.clearBuffer();
  display.setFont(u8g2_font_ncenB08_tr);

  // Ajusta a posição y para subir o título
  display.drawStr(10,10,"MENU TESTES");

  int numeroDeOpcoes = sizeof(menu_teste) / sizeof(char*);

  int alturaInicial = 20; // Você pode querer ajustar isso também, dependendo de quanto subiu o título
  char buffer[30];
  for (int i = 0; i < numeroDeOpcoes; i++)
  {
    int xOffset = 20; 
    strcpy_P(buffer, (char*)pgm_read_word(&(menu_teste[i])));
    if( i == contador_m_teste)
    {
      display.setFont(u8g2_font_open_iconic_arrow_1x_t);
      display.drawGlyph(20, alturaInicial + i*10, 0x0004e); // Ajuste a posição conforme necessário
      display.setFont(u8g2_font_ncenB08_tr);
      xOffset +=10;
    }
    display.drawStr(xOffset, alturaInicial + i*10, buffer);
  }
  display.sendBuffer();
}

void testeSensores(int led, int ldr)
{
  display.clearBuffer();
  display.setFont(u8g2_font_ncenB08_tr);
  display.drawStr(10,10,"TESTANDO...");
  display.sendBuffer();
  digitalWrite(led, HIGH); // Acende o LED
  delay(1000); // Dá tempo para a luz estabilizar e para o sensor reagir

  int valorMinimo = 700; // Inicia com 0, pois queremos encontrar o valor máximo

  unsigned long startTime = millis();
  unsigned long currentTime = startTime;

  // Enquanto não passar 2 segundos, lê o sensor e verifica se o valor é maior
  while ((currentTime - startTime) < 5000)
  {
    int leitura = analogRead(ldr);
    if (leitura <= valorMinimo && leitura > 600)
    {
      valorMinimo = leitura; // Atualiza o valor máximo se a nova leitura for maior
    }
    currentTime = millis(); // Atualiza o tempo atual
  }
  digitalWrite(led, LOW); // Desliga o LED após a leitura
  // Avalia o resultado baseado no valor máximo encontrado
  display.clearBuffer();
  display.setFont(u8g2_font_ncenB08_tr);
  if (valorMinimo < 700)
  {
    display.drawStr(0, 20, "Sensor abaixo de 700");
    display.drawStr(0,30,"Possível problema!");
    display.drawStr(0,45,"Press o btn direito!");
  } else if (valorMinimo == 1023)
  {
    display.drawStr(0, 20, "Valor maximo atingido");
    display.drawStr(0,30,"Possível problema!");
    display.drawStr(0,45,"Press o btn direito!");
  } else {
    display.drawStr(0, 20, "Sensor OK");
    display.drawStr(0,30,"Press o btn direito!");
    if(led != LED_1)
    {
      min_ldr = valorMinimo-100;//ajusta o novo limiar para o segundo ponto de passagem
    }
  }
    
  display.sendBuffer();
}

void print_msg_iniciando()
{
  unsigned long startMillis = millis();
  unsigned long currentMillis;
  int remainingTime = 10; // Contagem regressiva de 10 segundos

  while (remainingTime > 0)
  {
    currentMillis = millis();
    if (currentMillis - startMillis >= 1000)
    { // A cada 1 segundo
      startMillis = currentMillis;
      remainingTime--;

      display.clearBuffer();

      // Configura a fonte e exibe a mensagem
      display.setFont(u8g2_font_ncenB08_tr);
      display.drawStr(0, 10, "INICIANDO EM...");

      // Exibe os dígitos da contagem regressiva
      char remainingTimeString[4];
      sprintf(remainingTimeString, "%ds", remainingTime);
      display.setFont(u8g2_font_courR24_tf); // Fonte grande para os dígitos
      display.drawStr(50, 50, remainingTimeString); 

      display.sendBuffer();
    }
  }
    // Exibe a mensagem final
    display.clearBuffer();
    display.setFont(u8g2_font_ncenB08_tr); // Fonte menor para a mensagem final
    display.drawStr(15, 30, "Aguarde..."); 
    display.setFont(u8g2_font_5x7_tf);
    display.drawStr(0,45,"pss [enter] para cancelar!");
    display.setFont(u8g2_font_ncenB08_tr);
    display.sendBuffer();

    delay(1000); // Pausa antes de continuar
}

void print_concluido()
{

  display.clearBuffer();
  display.setFont(u8g2_font_ncenB08_tr);
  
  
  dentro_m_alturas = false;
  dentro_start = false;//saiu do iniciar
  dentro_m_dados = false;
  dentro_m_teste = false;
  dentro_m_principal = false;
  //delay(2000);
  if(!cancelar_captura && gravidade != 0)
  {
    display.drawStr(15,30,"CONCLUIDO");
    display.sendBuffer();
    delay(2000);
    contador_m_dados = 0;//isso 
    anterior_dados_in_contador = 0; // e isso, garatem que sempre que for concluido com sucesso vai para o menu dados
    dentro_m_dados = true;
    cancelar_captura = false;
    print_menu_dados();
   
  }else{
    display.drawStr(0,30,"CANCELANDO..");
    display.sendBuffer();
    delay(2000);
    contador_m_principal = 0;
    anterior_principal_in_contador = 0;
    dentro_m_principal = true;
    print_menu_principal();
  }
  
}

void print_gravidade()
{
  display.clearBuffer();
  display.setFont(u8g2_font_ncenB08_tr); // Fonte principal para "Gravidade em m/s"
  display.drawStr(0, 15, "Gravidade em m/s"); // Desenha a string principal

  display.setFont(u8g2_font_ncenB08_tr); // Define a fonte para o superíndice
  int x = display.getStrWidth("Gravidade em m/s"); // Calcula a largura do texto
  display.drawStr(x, 10, "2"); // Desenha o '2' como superíndice

  display.setFont(u8g2_font_ncenB08_tr); // Volta para a fonte principal
  display.drawStr(5, 35, str_gravidade); // Desenha o valor da gravidade
  display.sendBuffer();
}

void print_velocidade()
{
  display.clearBuffer();
  display.setFont(u8g2_font_ncenB08_tr);
  display.drawStr(0,10,"Velocidade em m/s");
  display.drawStr(5,20,str_velocidade);
  display.sendBuffer();
}
void print_erro()
{
  display.clearBuffer();
  display.setFont(u8g2_font_ncenB08_tr);
  display.drawStr(0,10,"Erro em %");
  display.drawStr(5,20,str_erro);
  display.sendBuffer();
}

void iniciar()//inicia o processo de captura
{
  if(index_altura == -1)//caso a altura não esteja selecionada
  {
    dentro_m_alturas = true;
    contador_m_alturas = 0;
    print_menu_alturas();

  }else{
  
    dentro_start = true; 
   // check = false;
    deltaTempo = 0.0;
    gravidade = 0.0;
    velocidade = gravidade;
    erro = velocidade;
    int led, ldr;
    //Apagando todos os leds

    digitalWrite(LED_1, LOW);
    digitalWrite(LED_2, LOW);
    digitalWrite(LED_3, LOW);
    digitalWrite(LED_4, LOW);

    if (index_altura == 0)
    {
      digitalWrite(LED_2, HIGH);
      led = LED_2;
      ldr = LDR_2;
    }
    if (index_altura == 1)
    {
      digitalWrite(LED_3, HIGH);
      led = LED_3;
      ldr = LDR_3;
    }
    if(index_altura == 2)
    {
      digitalWrite(LED_4,HIGH);
      led = LED_4;
      ldr = LDR_4;
    }
    digitalWrite(LED_1, HIGH);
    
    fechar_garra();
    print_msg_iniciando();
    capturar_tempos(led, ldr);
    print_concluido();
    
  }
}
void fechar_garra()//fecha a garra devagar
{
  for(int i = 0; i <= 45; i+=1)
  {
    delay(50);
    garra.write(i);
    
  }
}

void capturar_tempos(int l1, int l2)
{
  // Inicializa os tempos como zero
  tempo1 = 0;
  tempo2 = 0;
  check = false; //garantindo que o check será falso inicialmente
  cancelar_captura = false;
  garra.write(2); // abre a garra 

  while ((tempo1 == 0 || tempo2 == 0) && !cancelar_captura )
  {
    if (analogRead(LDR_1) > max_ldr )
    {
      tempo1 = micros();
      check = true;
      digitalWrite(LED_1, LOW); // Desliga o primeiro LED
    }
    while(check && !cancelar_captura)
    {
      if(analogRead(l2) < min_ldr)
      {
        tempo2 = micros();
        check = false;
        digitalWrite(l1, LOW);
      }
    }
  }

  if(tempo1 != 0 && tempo2 != 0)
  {      
    digitalWrite(l1,LOW);
    calcular();
  }
 
}
void calcular()/* está função calcula o tempo de queda, gravidade, velocidade de queda e o erro percentual*/
{
  deltaTempo = (tempo2 - tempo1)/1000000.0; //calcula a variação do tempo

  float d = alturas[index_altura];  // pega a altura selecionada

  gravidade = (2*d)/pow(deltaTempo,2); //calcula a gravidade com base na altura e no tempo
  velocidade = gravidade * deltaTempo; // velocidade calculada com base na gravidade e tempo, mas pode ser utilizado a distância e o tempo
  erro = ((abs(gravidade - g_referencial)) / g_referencial) * 100.0; //erro percentual.

  converter_para_string(); //converter para string
}

void converter_para_string()
{
  dtostrf(gravidade, 6, 2, str_gravidade);//converte em 4 casas decimais
  dtostrf(velocidade, 6, 2, str_velocidade);
  dtostrf(erro, 6, 2, str_erro);

  //adicionando as unidades
  snprintf(str_gravidade, sizeof(str_gravidade),"%sm/s^2",str_gravidade);
  snprintf(str_velocidade,sizeof(str_velocidade),"%sm/s",str_velocidade);
  snprintf(str_erro, sizeof(str_erro),"%s %%",str_erro);
}

void reset()/* cancela a captura  por ele na leitura do segundo sensor.*/
{
 cancelar_captura  = (dentro_start) ? false:true;

}
void voltar()
{
  print_menu_principal();
}
