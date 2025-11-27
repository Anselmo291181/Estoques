#include <stdio.h>      // biblioteca pra usar printf e scanf
#include <stdlib.h>     // pra usar system e exit
#include <string.h>     // pra usar strcmp e strcspn
#include <locale.h>     // pra usar acentos
#include <windows.h>    // pra funcionar as cores no windows

// Variaveis globais (aprendi que assim fica mais facil de usar em todo lugar)
const int MAX = 50; // maximo de 50 produtos
int ordenado = 0;   // guarda se ta ordenado ou nao
int ultimo_cod = 0; // guarda o ultimo codigo usado

// Cores que eu achei na internet pra deixar bonito
#define COR_RESET  "\x1b[0m"      // volta pra cor normal
#define COR_VERMELHO "\x1b[31m"   // cor vermelha
#define COR_VERDE "\x1b[32m"      // cor verde
#define COR_AMARELO "\x1b[33m"    // cor amarela
#define COR_CIANO "\x1b[36m"      // cor azul claro
#define COR_BRANCO "\x1b[37m"     // cor branca
#define COR_CINZA "\x1b[90m"      // cor cinza

// Isso aqui � pra n�o ter que escrever a cor toda hora
#define COR_CABECALHO COR_CIANO   // titulo usa ciano
#define COR_SUCESSO COR_VERDE     // mensagem de sucesso usa verde
#define COR_ERRO COR_VERMELHO     // mensagem de erro usa vermelho
#define COR_DADOS COR_BRANCO      // dados normais usa branco
#define COR_LINHA COR_CINZA       // linha usa cinza

// struct do produto (onde guardo as informacoes)
typedef struct {
long int codigo;      // codigo do produto
int grupo;            // numero do grupo
char descricao[41];   // nome do produto (ate 40 letras)
char unidade[3];      // unidade tipo kg, un, lt
char fornecedor[41];  // nome do fornecedor
float quantidade;     // quanto tem no estoque
float pr_compra;      // preco que eu comprei
float pr_venda;       // preco que eu vendo
int lucro;            // porcentagem de lucro
float estoque_min;    // estoque minimo (alerta)
} TProduto;

// funcoes (declarei aqui pra nao dar erro de compilacao)
void leitura(TProduto estoque[], int *tamanho);                    // le o arquivo
void gravacao(TProduto estoque[], int tamanho);                    // salva no arquivo
int pesquisabinaria(TProduto estoque[], int chave, int tamanho);   // busca por codigo
int pesquisastr(TProduto estoque[], int tamanho, char desc[]);     // busca por nome
int vazio(int tamanho);                                            // ve se ta vazio
void ordena(TProduto estoque[], int tamanho);                      // ordena por codigo
void inclusao(TProduto estoque[], int *tamanho);                   // cadastra produto novo
void relatorio_geral(TProduto estoque[], int tamanho);             // mostra todos os produtos
void excluir(TProduto estoque[], int *tamanho);                    // apaga produto
void ordena_pr_venda(TProduto estoque[],int tamanho);              // ordena por preco
void aumento(TProduto estoque[], int tamanho);                     // aumenta preco
void desconto(TProduto estoque[], int tamanho);                    // da desconto
void ordena_unidade(TProduto estoque[], int tamanho);              // ordena por unidade
void alerta_estoque_min(TProduto estoque[], int tamanho);          // alerta estoque baixo
void editar(TProduto estoque[], int tamanho);                      // edita produto
void fracionar_produto(TProduto estoque[], int *tamanho);		   //fraciona produtos
void printRel(TProduto estoque[], int posicao);                    // fun��o que guarda os elementos

//============================================================================

// fun��o pra ordenar usando bubble sort (aprendi na aula)
void ordena(TProduto estoque[], int tamanho) { 
int i, j;          // contadores do for
TProduto aux;      // variavel auxiliar pra fazer a troca

for (i = 0; i < tamanho - 1; i++) {              // percorre todo o vetor
for (j = i + 1; j < tamanho; j++) {          // compara com os proximos
if (estoque[i].codigo > estoque[j].codigo) { // se ta fora de ordem
aux = estoque[i];          // guarda o i
estoque[i] = estoque[j];   // coloca o j no lugar do i
estoque[j] = aux;          // coloca o i no lugar do j
}
}
}
}

int vazio(int tamanho) {  // funcao pra ver se ta vazio
return tamanho == 0;  // retorna 1 se for zero, 0 se nao for
}

// limpa o buffer (sempre esque�o de fazer isso e da bug)
void limpar_buffer() {
int c;                                               // variavel pra guardar o caracter
while ((c = getchar()) != '\n' && c != EOF) {}      // le ate achar enter ou fim
}

// essa fun��o ativa as cores no windows (copiei da internet)
void enableANSIColors() {
HANDLE hOut = GetStdHandle(STD_OUTPUT_HANDLE);   // pega o console
DWORD dwMode = 0;                                 // modo do console
GetConsoleMode(hOut, &dwMode);                    // le o modo atual
dwMode |= 0x0004;                                 // ativa as cores
SetConsoleMode(hOut, dwMode);                     // aplica o modo novo
}

// funcao pra fazer titulo bonito com linha
void printCabecalho(const char* texto) {
printf(COR_CABECALHO);                                                       // ativa a cor
printf("==============================================================================================\n"); // linha de cima
printf("\t\t\t\t%s\n", texto);                                              // imprime o titulo centralizado
printf("==============================================================================================\n\n"); // linha de baixo
printf(COR_RESET);                                                           // volta pra cor normal
}

void printLinha() {                                                              // funcao pra imprimir linha
printf(COR_LINHA);                                                           // ativa cor cinza
printf("==============================================================================================\n"); // imprime a linha
printf(COR_RESET);                                                           // volta pra cor normal
}

void printSucesso(const char* texto) {  // funcao pra mensagem de sucesso
printf(COR_SUCESSO);                // ativa cor verde
printf("%s\n", texto);              // imprime a mensagem
printf(COR_RESET);                  // volta pra cor normal
}

void printErro(const char* texto) {     // funcao pra mensagem de erro
printf(COR_ERRO);                   // ativa cor vermelha
printf("%s\n", texto);              // imprime a mensagem
printf(COR_RESET);                  // volta pra cor normal
}

void printNavegacao(const char* texto) { // funcao pra texto de navegacao
printf("%s", texto);                 // imprime o texto
printf(COR_RESET);                   // volta pra cor normal
}

//============================================================================

int main() {                                        // funcao principal do programa
setlocale(LC_ALL, "Portuguese");                // configura pra usar acentos
enableANSIColors();                             // ativa as cores

TProduto estoque[MAX];                          // vetor de produtos (ate 50)
int tamanho = 0;                                // quantidade de produtos cadastrados
int opcao;                                      // opcao escolhida no menu

leitura(estoque, &tamanho);                     // le os produtos do arquivo

do {                                            // loop do menu principal
printf(COR_BRANCO);                         // ativa cor branca
printf("______________________________________________________________________________________________\n"); // linha de cima
printf("\n          \t\t\t\t   MENU PRINCIPAL                \n");                                        // titulo do menu
printf("______________________________________________________________________________________________\n"); // linha de baixo

// fiz as cores intercaladas pra ficar legal
printf(COR_AMARELO "\n  1- CADASTRAR" COR_CINZA);               // opcao 1 em amarelo
printf(COR_CIANO "\n  2- RELATORIOS" COR_CINZA);                // opcao 2 em ciano
printf(COR_AMARELO "\n  3- BUSCA POR C�DIGO" COR_CINZA);        // opcao 3 em amarelo
printf(COR_CIANO "\n  4- BUSCA POR NOME" COR_CINZA);            // opcao 4 em ciano
printf(COR_AMARELO "\n  5- AUMENTO DE VALOR" COR_CINZA);        // opcao 5 em amarelo
printf(COR_CIANO "\n  6- DESCONTO DO VALOR" COR_CINZA);         // opcao 6 em ciano
printf(COR_AMARELO " \n  7- EDITAR PRODUTO" COR_CINZA);         // opcao 7 em amarelo
printf(COR_CIANO "\n  8- EXCLUIR PRODUTO" COR_CINZA);           // opcao 8 em ciano
printf(COR_AMARELO "\n  9- ALERTA ESTOQUE MINIMO" COR_CINZA);   // opcao 9 em amarelo
printf(COR_CIANO "\n 10- PICK");
printf(COR_CIANO "\n  0- SAIR" COR_CINZA);                      // opcao 0 pra sair
printf("\n______________________________________________________________________________________________\n"); // linha final
printf(COR_RESET);                                               // volta pra cor normal

printf("\nESCOLHA A OPCAO DESEJADA: ");                        // pede pra escolher
scanf("%d", &opcao);                                             // le a opcao
printf("\n\n");                                                  // pula 2 linhas
limpar_buffer();                                                 // limpa o buffer do teclado
system("cls");                                                   // limpa a tela

switch (opcao) {                                                 // verifica qual opcao foi escolhida
case 1:                                                      // se escolheu 1 (cadastrar)
inclusao(estoque, &tamanho);                             // chama funcao de cadastro
gravacao(estoque, tamanho);                              // salva no arquivo
break;                                                   // sai do switch

case 2: {                                                    // se escolheu 2 (relatorios)
int opRel;                                               // opcao do submenu
printf(COR_BRANCO);                                      // ativa cor branca
printf("\n______________________________________________________________________________________________\n"); // linha de cima
printf("\n        \t\t\t     MENU RELAT�RIOS                 ");                                        // titulo do submenu
printf("\n______________________________________________________________________________________________\n"); // linha de baixo
printf("  " COR_CIANO "\n1- RELAT�RIO GERAL" COR_CIANO);         // opcao 1 do submenu
printf("  " COR_AMARELO "\n2- RELAT�RIO POR PRE�O" COR_CIANO);    // opcao 2 do submenu
printf("  " COR_CIANO "\n3- RELAT�RIO POR UNIDADE" COR_CIANO );  // opcao 3 do submenu
printf("  " COR_BRANCO "\n0- VOLTAR" COR_CIANO);                  // opcao 0 pra voltar
printf("\n______________________________________________________________________________________________"); // linha final
printf(COR_RESET);                                                 // volta pra cor normal

printf("\nESCOLHA A OPCAO DESEJADA: ");                          // pede pra escolher
scanf("%d", &opRel);                                               // le a opcao
limpar_buffer();                                                   // limpa o buffer
system("cls");                                                     // limpa a tela

switch (opRel) {                                                   // verifica a opcao escolhida
case 1:                                                        // relatorio geral
relatorio_geral(estoque, tamanho);                         // chama a funcao
break;                                                     // sai do switch
case 2:                                                        // relatorio por preco
ordena_pr_venda(estoque, tamanho);                         // ordena por preco
relatorio_geral(estoque, tamanho);                         // mostra
break;                                                     // sai do switch
case 3:                                                        // relatorio por unidade
ordena_unidade(estoque, tamanho);                          // ordena por unidade
relatorio_geral(estoque, tamanho);                         // mostra
break;                                                     // sai do switch
case 0:                                                        // voltar
break;                                                     // sai do switch
default:                                                       // opcao invalida
printErro("OPCAO INVALIDA!");                              // mostra erro
printf("Pressione ENTER para continuar...");               // pede pra apertar enter
getchar();                                                 // espera apertar enter
break;                                                     // sai do switch
}
break;                                                             // sai do switch principal
}

case 3: { // BUSCA POR C�DIGO
long int codigo;
printf("Digite o c�digo do produto: ");
scanf("%ld", &codigo);
limpar_buffer();
int busca = pesquisabinaria(estoque, codigo, tamanho);  // busca bin�ria

if (busca != -1) {
// Mostra o produto encontrado
printLinha();

printRel(estoque, busca);  // mostra o produto

printLinha();

printSucesso("*** PRODUTO ENCONTRADO ***");  // confirma que achou
} else {
printErro("\n? Produto n�o encontrado.");  // n�o achou
printf("Verifique se o c�digo est� correto.\n");
}
printLinha();
printf("Pressione ENTER para continuar...");
getchar();
system("cls");
break;
}

case 4: {                                                          // busca por nome
char desc_busca[41];                                           // variavel pra guardar o nome
printf("Digite o nome do produto: ");                          // pede o nome
fgets(desc_busca, 41, stdin);                                  // le o nome
desc_busca[strcspn(desc_busca, "\n")] = 0;                     // tira o enter do final
int busca = pesquisastr(estoque, tamanho, desc_busca);         // busca o produto

if (busca != -1) {                                             // se achou
printSucesso("\nProduto encontrado:");                     // mostra mensagem de sucesso

printRel(estoque, busca);                                  // mostra o produto
} else {                                                       // se NAO achou
printErro("\n? Produto n�o encontrado.");                 // mensagem de erro
printf("Verifique se o nome est� correto.\n");             // sugere verificar
}

printf("Pressione ENTER para continuar...");                   // pede pra apertar enter
getchar();                                                     // espera
system("cls");                                                 // limpa tela
break;                                                         // sai do switch
}

case 5:                                                            // aumento de valor
aumento(estoque, tamanho);                                     // chama funcao de aumento
gravacao(estoque, tamanho);                                    // salva no arquivo
break;                                                         // sai do switch

case 6:                                                            // desconto do valor
desconto(estoque, tamanho);                                    // chama funcao de desconto
gravacao(estoque, tamanho);                                    // salva no arquivo
break;                                                         // sai do switch

case 7:                                                            // editar produto
editar(estoque, tamanho);                                      // chama funcao de edicao
gravacao(estoque, tamanho);                                    // salva no arquivo
break;                                                         // sai do switch

case 8:                                                            // excluir produto
excluir(estoque, &tamanho);                                    // chama funcao de exclusao
gravacao(estoque, tamanho);                                    // salva no arquivo
break;                                                         // sai do switch

case 9:                                                            // alerta estoque
alerta_estoque_min(estoque, tamanho);                          // chama funcao de alerta
gravacao(estoque, tamanho);                                    // salva no arquivo
break; 

case 10:                                                            
fracionar_produto(estoque,&tamanho);                        
gravacao(estoque, tamanho);                                     
break;   	                                                        // sai do switch

case 0:                                                            // sair do programa
printSucesso("Obrigado por usar nosso sistema.");              // mostra mensagem
printf("Pressione ENTER para sair...");                        // pede pra apertar enter
getchar();                                                     // espera
exit(0);                                                       // encerra o programa

default:                                                           // opcao invalida
printErro("OPCAO INVALIDA!");                                  // mostra erro
printf("Pressione ENTER para continuar...");                   // pede pra apertar enter
getchar();                                                     // espera
system("cls");                                                 // limpa tela
break;                                                         // sai do switch
}

} while (opcao != 0);                                                      // repete ate escolher 0

gravacao(estoque, tamanho);                                                // salva antes de fechar
return 0;                                                                  // retorna 0 (sucesso)
}

//============================================================================

void leitura(TProduto estoque[], int *tamanho) {                              // funcao pra ler o arquivo
FILE *arquivo;                                                             // ponteiro pro arquivo
arquivo = fopen("estoque.dat", "a+b");                                     // abre o arquivo (cria se nao existir)
if (!arquivo) {                                                            // se nao conseguiu abrir
printf("Erro ao abrir arquivo!");                                      // mostra erro
return;                                                                // sai da funcao
}

*tamanho = 0;                                                              // comeca com zero produtos
while(!feof(arquivo) && *tamanho < MAX) {                                  // enquanto nao chegar no fim e nao encher
if(fread(&estoque[*tamanho], sizeof(TProduto), 1, arquivo) == 1) {     // le 1 produto
(*tamanho)++;                                                      // aumenta o contador
}
}

// descobre qual o ultimo codigo usado (pra nao repetir)
ultimo_cod = 0;                                                            // comeca com zero
for(int i = 0; i < *tamanho; i++) {                                        // percorre todos os produtos
if(estoque[i].codigo > ultimo_cod) {                                   // se o codigo � maior
ultimo_cod = estoque[i].codigo;                                    // atualiza o ultimo codigo
}
}

fclose(arquivo);                                                           // fecha o arquivo
}

void gravacao(TProduto estoque[], int tamanho) {                              // funcao pra salvar no arquivo
FILE *arquivo;                                                             // ponteiro pro arquivo
arquivo = fopen("estoque.dat", "w+b");                                     // abre o arquivo (apaga o conteudo antigo)
if (!arquivo) {                                                            // se nao conseguiu abrir
printf("Erro ao abrir arquivo!");                                      // mostra erro
return;                                                                // sai da funcao
}

for(int i = 0; i < tamanho; i++) {                                         // percorre todos os produtos
fwrite(&estoque[i], sizeof(TProduto), 1, arquivo);                     // escreve 1 produto no arquivo
}

fclose(arquivo);                                                           // fecha o arquivo
}

//============================================================================
// cadastrar produto novo
void inclusao(TProduto estoque[], int *tamanho) {                             // funcao de cadastro
if (*tamanho == MAX) {                                                     // se ja ta cheio (50 produtos)
printErro("\n ERRO! \n ARQUIVO CHEIO.\n");                             // mostra erro
return;                                                                // sai da funcao
}

TProduto aux;                                                              // produto temporario
char correto = 'n';                                                        // variavel pra confirmacao

// gera o codigo automatico
int codigo = ++ultimo_cod;                                                 // incrementa e pega o proximo codigo
aux.codigo = codigo;                                                       // atribui ao produto

printf(COR_AMARELO "C�digo: %ld\n" COR_RESET, aux.codigo);                // mostra o codigo gerado

do {                                                                        // loop ate digitar certo
printf(COR_CIANO "Grupo:..............................: " COR_RESET);     // pede o grupo
printf(COR_BRANCO);                                                        // muda a cor
scanf("%i", &aux.grupo);                                                   // le o grupo
printf(COR_RESET);                                                         // volta a cor normal
limpar_buffer();                                                           // limpa o buffer
} while (aux.grupo <= 0);                                                      // repete se for menor ou igual a zero

do {                                                                           // loop ate digitar certo
printf("\n" COR_AMARELO "Descri��o:...........................: " COR_RESET); // pede a descricao
fgets(aux.descricao, 41, stdin);                                           // le a descricao
aux.descricao[strcspn(aux.descricao, "\n")] = 0;                           // tira o enter do final
} while (strlen(aux.descricao) == 0);                                          // repete se estiver vazio

do {                                                                           // loop ate digitar certo
printf("\n" COR_CIANO "Unidade .............................: "COR_RESET); // pede a unidade
fgets(aux.unidade, 3, stdin);                                              // le a unidade
aux.unidade[strcspn(aux.unidade, "\n")] = 0;                               // tira o enter
limpar_buffer();                                                          // limpa o buffer
} while (strlen(aux.unidade) == 0);                                            // repete se estiver vazio

do {                                                                           // loop ate digitar certo
printf("\n" COR_AMARELO "Fornecedor ..........................: " COR_RESET); // pede o fornecedor
fgets(aux.fornecedor, 41, stdin);                                          // le o fornecedor
aux.fornecedor[strcspn(aux.fornecedor, "\n")] = 0;                         // tira o enter
limpar_buffer();                                                          // limpa o buffer
} while (strlen(aux.fornecedor) == 0);                                         // repete se estiver vazio

do {                                                                           // loop ate digitar certo
printf("\n" COR_CIANO "Quantidade:..........................: " COR_RESET); // pede a quantidade
scanf("%f", &aux.quantidade);                                              // le a quantidade
limpar_buffer();                                                           // limpa o buffer
} while(aux.quantidade < 0);                                                   // repete se for negativo

do {                                                                           // loop ate digitar certo
printf("\n" COR_AMARELO "Pre�o de Compra:.....................: " COR_RESET); // pede o preco de compra
scanf("%f", &aux.pr_compra);                                               // le o preco
limpar_buffer();                                                           // limpa o buffer
} while(aux.pr_compra < 0);                                                    // repete se for negativo

do {                                                                           // loop ate digitar certo
printf("\n" COR_CIANO "Pre�o de Venda:......................: " COR_RESET); // pede o preco de venda
scanf("%f", &aux.pr_venda);                                                // le o preco
limpar_buffer();                                                           // limpa o buffer
} while(aux.pr_venda < 0);                                                     // repete se for negativo

do {                                                                           // loop ate digitar certo
printf("\n" COR_AMARELO "% de Lucro ...........................: " COR_RESET); // pede o lucro
scanf("%i", &aux.lucro);                                                   // le o lucro
limpar_buffer();                                                           // limpa o buffer
} while(aux.lucro < 0);                                                        // repete se for negativo

do {                                                                           // loop ate digitar certo
printf("\n" COR_CIANO "Estoque Minimo ......................: "COR_RESET); // pede o estoque minimo
scanf("%f", &aux.estoque_min);                                             // le o estoque minimo
limpar_buffer();                                                           // limpa o buffer
} while(aux.estoque_min < 0);                                                  // repete se for negativo

printf(COR_AMARELO "Os dados estao corretos?(S/N)" COR_RESET);                // pergunta se ta certo
correto = getchar();                                                           // le S ou N
limpar_buffer();                                                               // limpa o buffer
system("cls");                                                                 // limpa a tela

if (correto=='s'||correto=='S'){                                           // se confirmou
estoque [*tamanho] = aux;                                              // copia o produto pro vetor
(*tamanho) ++;                                                         // aumenta o tamanho
ordenado=0;                                                            // marca que nao ta mais ordenado
printSucesso ("\tO PRODUTO FOI INCLUIDO!");                            // mostra mensagem de sucesso
} else {                                                                   // se nao confirmou
printf("\tCADASTRO CANCELADO!");                                       // mostra que cancelou
ultimo_cod--;                                                          // volta o codigo (nao foi usado)
}

printf("\tAPERTE ENTER PARA VOLTAR AO MENU");                              // pede pra apertar enter
getchar();                                                                 // espera
system("cls");                                                             // limpa a tela
}

//============================================================================
// relatorio principal
void relatorio_geral(TProduto estoque[], int tamanho) {                       // funcao do relatorio
int pagina_atual = 0;                                                      // comeca na pagina 0
int produtos_por_pagina = 2;                                               // mostra 2 produtos por pagina
int total_paginas = (tamanho + produtos_por_pagina - 1) / produtos_por_pagina; // calcula total de paginas
int opcao;                                                                 // opcao de navegacao

if (tamanho == 0) {                                                        // se nao tem nenhum produto
printf("Nenhum produto cadastrado para exibir.");                      // mostra mensagem
printf("APERTE ENTER PARA VOLTAR AO MENU\n");                          // pede pra voltar
getchar();                                                             // espera
return;                                                                // sai da funcao
}

ordena(estoque, tamanho);                                                  // ordena antes de mostrar

do {                                                                       // loop de navegacao
// limpa a tela (jeito que eu achei que funciona)
for(int i = 0; i < 50; i++) {                                          // loop de 50 vezes
printf("\n");                                                      // imprime linha vazia
}

printCabecalho("Controle de Estoque - Relat�rio Geral");              // imprime o titulo

int inicio = pagina_atual * produtos_por_pagina;                       // calcula o primeiro produto
int fim = inicio + produtos_por_pagina;                                // calcula o ultimo produto
if (fim > tamanho) fim = tamanho;                                      // ajusta se passar do total

for (int i = inicio; i < fim; i++) {                                   // percorre os produtos da pagina
printLinha();                                                      // linha antes do produto
printRel(estoque, i);                                              // CORRE��O: mostra o produto na posi��o i
printLinha();                                                      // linha depois do produto

if (i < fim - 1) {                                                 // se nao � o ultimo
printf("\n");                                                  // pula linha entre produtos
}
}

printLinha();                                                          // imprime linha
printf(COR_AMARELO "P�gina %d de %d\n" COR_RESET, pagina_atual + 1, total_paginas); // mostra pagina atual
printNavegacao("NAVEGA��O: \n");                                       // mostra texto de navegacao
if (pagina_atual > 0) printf(COR_VERDE "[1] Anterior " COR_RESET);    // mostra opcao anterior se nao � a primeira
if (pagina_atual < total_paginas - 1) printf(COR_VERDE "[2] Pr�xima " COR_RESET); // mostra opcao proxima se nao � a ultima
printf(COR_VERMELHO "[0] Sair\n" COR_RESET);                           // mostra opcao sair
printf("Escolha: ");                                                   // pede pra escolher

scanf("%d", &opcao);                                                   // le a opcao
while (getchar() != '\n');                                             // limpa o buffer completamente

if (opcao == 1 && pagina_atual > 0) {                                  // se escolheu anterior e pode voltar
pagina_atual--;                                                    // volta uma pagina
} else if (opcao == 2 && pagina_atual < total_paginas - 1) {           // se escolheu proxima e pode avancar
pagina_atual++;                                                    // avanca uma pagina
} else if (opcao == 0) {                                               // se escolheu sair
break;                                                             // sai do loop
} else {                                                               // se opcao invalida
printErro("Op��o inv�lida! Pressione ENTER...");                   // mostra erro
getchar();                                                         // espera
}

} while (opcao != 0);                                                      // repete ate escolher sair

for(int i = 0; i < 50; i++) {                                              // limpa a tela no final
printf("\n");                                                          // imprime linha vazia
}
}

//============================================================================

void ordena_pr_venda(TProduto estoque[], int tamanho) {                       // funcao pra ordenar por preco
int pagina_atual = 0;                                                      // pagina inicial
int produtos_por_pagina = 15;                                              // 15 produtos por pagina
int total_paginas = (tamanho + produtos_por_pagina - 1) / produtos_por_pagina; // calcula paginas
int opcao;                                                                 // opcao de navegacao
int i, j;                                                                  // contadores
TProduto aux;                                                              // produto auxiliar

// ordena por preco (bubble sort)
for (i = 0; i < tamanho - 1; i++) {                                        // percorre o vetor
for (j = i + 1; j < tamanho; j++) {                                    // compara com os proximos
if (estoque[i].pr_venda > estoque[j].pr_venda) {                   // se ta fora de ordem
aux = estoque[i];                                              // guarda o i
estoque[i] = estoque[j];                                       // coloca j no lugar de i
estoque[j] = aux;                                              // coloca i no lugar de j
}
}
}


if (tamanho == 0) {                                                        // se nao tem produtos
printf("Nenhum produto cadastrado para exibir.");                      // mostra mensagem
printf("APERTE ENTER PARA VOLTAR AO MENU\n");                          // pede pra voltar
getchar();                                                             // espera
return;                                                                // sai da funcao
}

do {                                                                       // loop de navegacao
for(int i = 0; i < 50; i++) {                                          // limpa a tela
printf("\n");                                                      // imprime linha vazia
}

printCabecalho("Controle de Estoque - Relat�rio Por Pre�o");          // imprime titulo
printf(COR_BRANCO "%-30s %-55s %-10s\n" COR_RESET, "C�digo", "Descri��o", "Pre�o\n"); // cabecalho da tabela

int inicio = pagina_atual * produtos_por_pagina;                       // calcula primeiro produto
int fim = inicio + produtos_por_pagina;                                // calcula ultimo produto
if (fim > tamanho) fim = tamanho;                                      // ajusta se passar do total

for (int i = inicio; i < fim; i++) {                                       // percorre os produtos
if (i % 2 == 0) {                                                          // se linha par
// linha par - amarelo (pra intercalar as cores)
printf(COR_AMARELO "%-32ld" COR_RESET, estoque[i].codigo);             // mostra codigo
printf(COR_AMARELO "%-52s" COR_RESET, estoque[i].descricao);           // mostra descricao
printf(COR_AMARELO "%10.2f\n" COR_RESET, estoque[i].pr_venda);         // mostra preco
} else {                                                                   // se linha impar
// linha impar - ciano
printf(COR_CIANO "%-32ld" COR_RESET, estoque[i].codigo);               // mostra codigo
printf(COR_CIANO "%-52s" COR_RESET, estoque[i].descricao);             // mostra descricao
printf(COR_CIANO "%10.2f\n" COR_RESET, estoque[i].pr_venda);           // mostra preco
}
}


printf(COR_BRANCO "\nP�gina %d de %d" COR_RESET, pagina_atual + 1, total_paginas); // mostra pagina atual

printf("\n");                                                          // pula linha
printLinha();                                                          // imprime linha
printNavegacao("NAVEGA��O: ");                                         // texto de navegacao
if (pagina_atual > 0) printf(COR_VERDE "[1] Anterior " COR_RESET);    // botao anterior
if (pagina_atual < total_paginas - 1) printf(COR_VERDE "[2] Pr�xima " COR_RESET); // botao proxima
printf(COR_VERMELHO "[0] Sair\n" COR_RESET);                           // botao sair
printf("Escolha: ");                                                   // pede pra escolher

scanf("%d", &opcao);                                                   // le a opcao
while (getchar() != '\n');                                             // limpa o buffer

if (opcao == 1 && pagina_atual > 0) {                                  // se pode voltar
pagina_atual--;                                                    // volta pagina
} else if (opcao == 2 && pagina_atual < total_paginas - 1) {           // se pode avancar
pagina_atual++;                                                    // avanca pagina
} else if (opcao == 0) {                                               // se escolheu sair
break;                                                             // sai do loop
} else {                                                               // se opcao invalida
printErro("Op��o inv�lida! Pressione ENTER...");                   // mostra erro
getchar();                                                         // espera
}

} while (opcao != 0);                                                      // repete ate sair

for(int i = 0; i < 50; i++) {                                              // limpa a tela no final
printf("\n");                                                          // imprime linha vazia
}
}

//============================================================================

void ordena_unidade(TProduto estoque[], int tamanho) {                        // funcao pra ordenar por unidade

int pagina_atual = 0;                                                      // pagina inicial
int produtos_por_pagina = 2;                                               // 2 produtos por pagina
int total_paginas = (tamanho + produtos_por_pagina - 1) / produtos_por_pagina; // calcula total de paginas
int opcao;                                                                 // opcao de navegacao

// ordena por unidade (strcmp compara strings em ordem alfabetica)
for (int i = 0; i < tamanho - 1; i++) {                                    // percorre o vetor
for (int j = i + 1; j < tamanho; j++) {                                // compara com os proximos
if (strcmp(estoque[i].unidade, estoque[j].unidade) > 0) {          // se ta fora de ordem
TProduto aux = estoque[i];                                     // guarda o i
estoque[i] = estoque[j];                                       // coloca j no lugar de i
estoque[j] = aux;                                              // coloca i no lugar de j
}
}
}

if (tamanho == 0) {                                                        // se nao tem produtos
printf("Nenhum produto cadastrado para exibir.");                      // mostra mensagem
printf("APERTE ENTER PARA VOLTAR AO MENU\n");                          // pede pra voltar
getchar();                                                             // espera
return;                                                                // sai da funcao
}

do {                                                                       // loop de navegacao
for(int i = 0; i < 50; i++) printf("\n");                              // limpa a tela

printCabecalho("Controle de Estoque - Relat�rio por Unidade");        // imprime titulo

int inicio = pagina_atual * produtos_por_pagina;                       // calcula primeiro produto
int fim = inicio + produtos_por_pagina;                                // calcula ultimo produto
if (fim > tamanho) fim = tamanho;                                      // ajusta se passar do total

for (int i = inicio; i < fim; i++) {                                  // percorre os produtos
printf(COR_CIANO "C�digo:" COR_BRANCO " %-33ld" COR_RESET, estoque[i].codigo); // mostra codigo
printf("%*s", 45, "");                                                 // espacamento
printf(COR_CIANO "Grupo:" COR_BRANCO " %d\n" COR_RESET, estoque[i].grupo); // mostra grupo

printf(COR_CIANO "Descri��o:" COR_BRANCO " %-62s" COR_RESET, estoque[i].descricao); // mostra descricao
printf("%*s", 10, "");                                                 // espacamento
printf(COR_CIANO "Unidade:" COR_BRANCO " %s\n" COR_RESET, estoque[i].unidade); // mostra unidade

printf(COR_CIANO "Fornecedor:" COR_BRANCO " %s\n" COR_RESET, estoque[i].fornecedor); // mostra fornecedor

printf(COR_CIANO "Pre�o de Compra:" COR_BRANCO " %-20.2f" COR_RESET, estoque[i].pr_compra); // mostra preco compra
printf("%*s", 5, "");                                                  // espacamento

printf(COR_CIANO "Pre�o de Venda:" COR_BRANCO " %-14.2f" COR_RESET, estoque[i].pr_venda); // mostra preco venda
printf("%*s", 5, "");                                                  // espacamento

printf(COR_CIANO "Lucro M�nimo:" COR_BRANCO " %d%%\n" COR_RESET, estoque[i].lucro); // mostra lucro

printf(COR_CIANO "Quantidade:" COR_BRANCO " %-33.0f" COR_RESET, estoque[i].quantidade); // mostra quantidade
printf("%*s", 25, "");                                                 // espacamento
printf(COR_CIANO "Quantidade M�nima:" COR_BRANCO " %.2f\n" COR_RESET, estoque[i].estoque_min); // mostra estoque min

if (i < fim - 1) {                                                 // se nao � o ultimo
printLinha();                                                  // imprime linha divisoria
}
}

printf(COR_CIANO "\nP�gina %d de %d\n" COR_RESET, pagina_atual + 1, total_paginas); // mostra pagina atual
printLinha();                                                          // imprime linha
printNavegacao("NAVEGA��O: ");                                         // texto de navegacao
if (pagina_atual > 0) printf(COR_VERDE "[1] Anterior " COR_RESET);    // botao anterior
if (pagina_atual < total_paginas - 1) printf(COR_VERDE "[2] Pr�xima " COR_RESET); // botao proxima
printf(COR_VERMELHO "[0] Sair\n" COR_RESET);                           // botao sair
printf("Escolha: ");                                                   // pede pra escolher

scanf("%d", &opcao);                                                   // le a opcao
while (getchar() != '\n');                                             // limpa o buffer

if (opcao == 1 && pagina_atual > 0) pagina_atual--;                    // volta pagina se pode
else if (opcao == 2 && pagina_atual < total_paginas - 1) pagina_atual++; // avanca pagina se pode
else if (opcao == 0) break;                                            // sai do loop
else {                                                                 // se opcao invalida
printErro("Op��o inv�lida! ENTER...");                             // mostra erro
getchar();                                                         // espera
}

} while (opcao != 0);                                                      // repete ate sair

for(int i = 0; i < 50; i++) printf("\n");                                  // limpa a tela no final
}

//============================================================================
// busca binaria (mais rapida que busca normal porque divide ao meio)
int pesquisabinaria(TProduto estoque[], int chave, int tamanho) {             // funcao de busca binaria
if(vazio(tamanho)) return -1;                                              // se ta vazio retorna -1
if (!ordenado) {                                                           // se nao ta ordenado
ordena(estoque, tamanho);                                              // ordena primeiro
ordenado = 1;                                                          // marca como ordenado
}

int inicio = 0, final = tamanho - 1, meio;                                 // variaveis de controle
while (inicio <= final) {                                                  // enquanto tiver elementos pra procurar
meio = (inicio + final) / 2;                                           // calcula o meio
if (estoque[meio].codigo == chave) return meio;                        // se achou retorna a posicao
if (estoque[meio].codigo < chave) inicio = meio + 1;                   // procura na metade direita
else final = meio - 1;                                                 // procura na metade esquerda
}
return -1;                                                                 // nao achou retorna -1
}

//============================================================================
// busca por nome (busca sequencial, procura um por um)
int pesquisastr(TProduto estoque[], int tamanho, char desc[]) {               // funcao de busca por nome
for (int i = 0; i < tamanho; i++) {                                        // percorre todos os produtos
if (strcmp(estoque[i].descricao, desc) == 0) return i;                 // se achou retorna a posicao
}
return -1;                                                                 // nao achou retorna -1
}

//============================================================================
// aumenta preco de um grupo inteiro
void aumento(TProduto estoque[], int tamanho) {                               // funcao de aumento
int grupo, porcentagem;                                                    // variaveis
printf("Informe o grupo que deseja reajustar: ");                          // pede o grupo
scanf("%d", &grupo);                                                       // le o grupo
limpar_buffer();                                                           // limpa o buffer
printf("Informe a porcentagem de aumento: ");                              // pede a porcentagem
scanf("%d", &porcentagem);                                                 // le a porcentagem
limpar_buffer();                                                           // limpa o buffer

float fator = 1 + (porcentagem / 100.0);                                   // calcula o fator (ex: 1.10 pra 10%)
int alterou = 0;                                                           // flag pra saber se alterou algum
int preco_zero = 0;                                                        // contador de precos zero

printCabecalho("PRODUTOS REAJUSTADOS");                                    // imprime titulo
printf(COR_AMARELO "\t\t\t\tGRUPO: %d | AUMENTO: %d%%\n\n" COR_RESET, grupo, porcentagem); // mostra info

for (int i = 0; i < tamanho; i++) {                                        // percorre todos os produtos
if (estoque[i].grupo == grupo) {                                       // se � do grupo escolhido
float preco_antigo = estoque[i].pr_venda;                          // guarda o preco antigo

if (preco_antigo <= 0) {                                           // se preco � zero ou negativo
preco_zero++;                                                  // conta mais um
continue;                                                      // pula pro proximo
}

estoque[i].pr_venda = preco_antigo * fator;                        // aplica o aumento
alterou = 1;                                                       // marca que alterou

printf(COR_VERDE "C�digo: %-30ld" COR_RESET, estoque[i].codigo);   // mostra codigo
printf("%*s", 45, "");                                             // espacamento
printf(COR_AMARELO "Grupo: %d\n" COR_RESET, estoque[i].grupo);     // mostra grupo

printf(COR_DADOS "Descri��o: %-62s" COR_RESET, estoque[i].descricao); // mostra descricao
printf("%*s", 10, "");                                             // espacamento
printf(COR_CIANO "Unidade: %s\n" COR_RESET, estoque[i].unidade);   // mostra unidade

printf(COR_DADOS "Fornecedor: %s\n" COR_RESET, estoque[i].fornecedor); // mostra fornecedor

printf(COR_AMARELO "Pre�o Anterior: %-20.2f" COR_RESET, preco_antigo); // mostra preco antigo
printf("%*s", 5, "");                                              // espacamento
printf(COR_VERDE "Novo Pre�o: %-24.2f" COR_RESET, estoque[i].pr_venda); // mostra preco novo
printf("%*s", 5, "");                                              // espacamento
printf(COR_AMARELO "Aumento: %d%%\n" COR_RESET, porcentagem);      // mostra porcentagem

printf(COR_CIANO "Quantidade: %-33.0f" COR_RESET, estoque[i].quantidade); // mostra quantidade
printf("%*s", 25, "");                                             // espacamento
printf(COR_AMARELO "Quantidade M�nima: %.2f\n" COR_RESET, estoque[i].estoque_min); // mostra estoque min

printLinha();                                                      // imprime linha
printf("\n");                                                      // pula linha
}
}

if (alterou) {                                                             // se alterou pelo menos um
printSucesso("Reajuste aplicado com sucesso!");                        // mostra mensagem de sucesso
} else {                                                                   // se nao alterou nenhum
printErro("Nenhum produto v�lido encontrado neste grupo.");            // mostra erro
}

if (preco_zero > 0) {                                                      // se teve produtos com preco zero
printf("AVISO: %d produto(s) ignorado(s) por terem pre�o zero.", preco_zero); // mostra aviso
}

printf("Pressione ENTER para continuar...");                               // pede pra continuar
getchar();                                                                 // espera
}

//============================================================================
// da desconto no preco de um grupo
void desconto(TProduto estoque[], int tamanho) {                              // funcao de desconto
int grupo, porcentagem;                                                    // variaveis
printf("Informe o grupo que deseja dar desconto: ");                          // pede o grupo
scanf("%d", &grupo);                                                       // le o grupo
limpar_buffer();                                                           // limpa o buffer
printf("Informe a porcentagem de desconto: ");                             // pede a porcentagem
scanf("%d", &porcentagem);                                                 // le a porcentagem
limpar_buffer();                                                           // limpa o buffer

float fator = 1 - (porcentagem / 100.0);                                   // calcula o fator (ex: 0.90 pra 10% off)
int alterou = 0;                                                           // flag pra saber se alterou
int preco_zero = 0;                                                        // contador de precos zero

printCabecalho("PRODUTOS COM DESCONTO");                                   // imprime titulo
printf(COR_AMARELO "\t\t\t\tGRUPO: %d | DESCONTO: %d%%\n\n" COR_RESET, grupo, porcentagem); // mostra info

for (int i = 0; i < tamanho; i++) {                                        // percorre todos os produtos
if (estoque[i].grupo == grupo) {                                       // se � do grupo escolhido
float preco_antigo = estoque[i].pr_venda;                          // guarda o preco antigo

if (preco_antigo <= 0) {                                           // se preco � zero ou negativo
preco_zero++;                                                  // conta mais um
continue;                                                      // pula pro proximo
}

estoque[i].pr_venda = preco_antigo * fator;                        // aplica o desconto
alterou = 1;                                                       // marca que alterou

printf(COR_VERDE "C�digo: %-30ld" COR_RESET, estoque[i].codigo);   // mostra codigo
printf("%*s", 45, "");                                             // espacamento
printf(COR_VERDE "Grupo: %d\n" COR_RESET, estoque[i].grupo);       // mostra grupo

printf(COR_DADOS "Descri��o: %-62s" COR_RESET, estoque[i].descricao); // mostra descricao
printf("%*s", 10, "");                                             // espacamento
printf(COR_CIANO "Unidade: %s\n" COR_RESET, estoque[i].unidade);   // mostra unidade

printf(COR_DADOS "Fornecedor: %s\n" COR_RESET, estoque[i].fornecedor); // mostra fornecedor

printf(COR_AMARELO "Pre�o Anterior: %-20.2f" COR_RESET, preco_antigo); // mostra preco antigo
printf("%*s", 5, "");                                              // espacamento
printf(COR_VERDE "Novo Pre�o: %-24.2f" COR_RESET, estoque[i].pr_venda); // mostra preco novo
printf("%*s", 5, "");                                              // espacamento
printf(COR_AMARELO "Desconto: %d%%\n" COR_RESET, porcentagem);     // mostra porcentagem

printf(COR_CIANO "Quantidade: %-33.0f" COR_RESET, estoque[i].quantidade); // mostra quantidade
printf("%*s", 25, "");                                             // espacamento
printf(COR_AMARELO "Quantidade M�nima: %.2f\n" COR_RESET, estoque[i].estoque_min); // mostra estoque min

printLinha();                                                      // imprime linha
printf("\n");                                                      // pula linha
}
}

if (alterou) {                                                             // se alterou pelo menos um
printSucesso("Desconto aplicado com sucesso!");                        // mostra mensagem de sucesso
} else {                                                                   // se nao alterou nenhum
printErro("Nenhum produto v�lido encontrado neste grupo.");            // mostra erro
}

if (preco_zero > 0) {                                                      // se teve produtos com preco zero
printf(COR_VERMELHO "AVISO: %d produto(s) ignorado(s) por terem pre�o zero.\n" COR_RESET, preco_zero); // mostra aviso
}

printf("Pressione ENTER para continuar...");                               // pede pra continuar
getchar();                                                                 // espera
}

//============================================================================
// editar produto (pode mudar qualquer informacao)
void editar(TProduto estoque[], int tamanho) {                                // funcao de edicao

if (tamanho == 0) {                                                        // se nao tem produtos
printf("Nenhum produto cadastrado.");                                  // mostra mensagem
printf("Pressione ENTER para continuar...");                           // pede pra continuar
getchar();                                                             // espera
return;                                                                // sai da funcao
}

long int codigo;                                                           // variavel pro codigo
printf("Digite o c�digo do produto a editar: ");                           // pede o codigo
scanf("%ld", &codigo);                                                     // le o codigo
limpar_buffer();                                                           // limpa o buffer

int posicao = pesquisabinaria(estoque, codigo, tamanho);                   // busca o produto
if (posicao == -1) {                                                       // se nao achou
printErro("Produto n�o encontrado.");                                  // mostra erro
printf("Pressione ENTER para continuar...");                           // pede pra continuar
getchar();                                                             // espera
return;                                                                // sai da funcao
}

TProduto aux = estoque[posicao];                                           // copia o produto pra uma variavel temporaria
char opcao;                                                                // variavel pra S ou N

printf(COR_VERDE "\nProduto encontrado:\n" COR_RESET);                     // mostra mensagem
printf(COR_DADOS "C�digo: %ld\nDescri��o: %s\nPre�o: %.2f\nQuantidade: %.2f\n" COR_RESET, // mostra os dados atuais
aux.codigo, aux.descricao, aux.pr_venda, aux.quantidade);

// pergunta campo por campo se quer mudar (o usuario escolhe o que editar)
printf(COR_AMARELO "\nDeseja alterar o grupo? (S/N): " COR_RESET);        // pergunta do grupo
opcao = getchar(); limpar_buffer();                                        // le S ou N
if (opcao == 'S' || opcao == 's') {                                        // se quis alterar
printf("Novo grupo: ");                                                // pede o novo grupo
scanf("%d", &aux.grupo);                                               // le o novo grupo
limpar_buffer();                                                       // limpa o buffer
}

printf(COR_AMARELO "Deseja alterar a descri��o? (S/N): " COR_RESET);      // pergunta da descricao
opcao = getchar(); limpar_buffer();                                        // le S ou N
if (opcao == 'S' || opcao == 's') {                                        // se quis alterar
printf("Nova descri��o: ");                                            // pede a nova descricao
fgets(aux.descricao, 41, stdin);                                       // le a nova descricao
aux.descricao[strcspn(aux.descricao, "\n")] = 0;                       // tira o enter
}

printf(COR_AMARELO "Deseja alterar a unidade? (S/N): " COR_RESET);        // pergunta da unidade
opcao = getchar(); limpar_buffer();                                        // le S ou N
if (opcao == 'S' || opcao == 's') {                                        // se quis alterar
printf("Nova unidade: ");                                              // pede a nova unidade
fgets(aux.unidade, 3, stdin);                                          // le a nova unidade
aux.unidade[strcspn(aux.unidade, "\n")] = 0;                           // tira o enter
}

printf(COR_AMARELO "Deseja alterar o fornecedor? (S/N): " COR_RESET);     // pergunta do fornecedor
opcao = getchar(); limpar_buffer();                                        // le S ou N
if (opcao == 'S' || opcao == 's') {                                        // se quis alterar
printf("Novo fornecedor: ");                                           // pede o novo fornecedor
fgets(aux.fornecedor, 41, stdin);                                      // le o novo fornecedor
aux.fornecedor[strcspn(aux.fornecedor, "\n")] = 0;                     // tira o enter
}

printf(COR_AMARELO "Deseja alterar a quantidade? (S/N): " COR_RESET);     // pergunta da quantidade
opcao = getchar(); limpar_buffer();                                        // le S ou N
if (opcao == 'S' || opcao == 's') {                                        // se quis alterar
printf("Nova quantidade: ");                                           // pede a nova quantidade
scanf("%f", &aux.quantidade);                                          // le a nova quantidade
limpar_buffer();                                                       // limpa o buffer
}

printf(COR_AMARELO "Deseja alterar o pre�o de compra? (S/N): " COR_RESET); // pergunta do preco de compra
opcao = getchar(); limpar_buffer();                                        // le S ou N
if (opcao == 'S' || opcao == 's') {                                        // se quis alterar
printf("Novo pre�o de compra: ");                                      // pede o novo preco
scanf("%f", &aux.pr_compra);                                           // le o novo preco
limpar_buffer();                                                       // limpa o buffer
}

printf(COR_AMARELO "Deseja alterar o pre�o de venda? (S/N): " COR_RESET); // pergunta do preco de venda
opcao = getchar(); limpar_buffer();                                        // le S ou N
if (opcao == 'S' || opcao == 's') {                                        // se quis alterar
printf("Novo pre�o de venda: ");                                       // pede o novo preco
scanf("%f", &aux.pr_venda);                                            // le o novo preco
limpar_buffer();                                                       // limpa o buffer
}

printf(COR_AMARELO "Deseja alterar o lucro m�nimo? (S/N): " COR_RESET);   // pergunta do lucro
opcao = getchar(); limpar_buffer();                                        // le S ou N
if (opcao == 'S' || opcao == 's') {                                        // se quis alterar
printf("Novo lucro m�nimo (%%): ");                                    // pede o novo lucro
scanf("%d", &aux.lucro);                                               // le o novo lucro
limpar_buffer();                                                       // limpa o buffer
}

printf(COR_AMARELO "Deseja alterar o estoque m�nimo? (S/N): " COR_RESET); // pergunta do estoque minimo
opcao = getchar(); limpar_buffer();                                        // le S ou N
if (opcao == 'S' || opcao == 's') {                                        // se quis alterar
printf("Novo estoque m�nimo: ");                                       // pede o novo estoque
scanf("%f", &aux.estoque_min);                                         // le o novo estoque
limpar_buffer();                                                       // limpa o buffer
}

estoque[posicao] = aux;                                                    // salva as alteracoes no vetor
printSucesso("\nProduto editado com sucesso!");                            // mostra mensagem de sucesso
printf("Pressione ENTER para continuar...");                               // pede pra continuar
getchar();                                                                 // espera
}

//============================================================================
// excluir produto (remove do vetor)
void excluir(TProduto estoque[], int *tamanho) {                              // funcao de exclusao
if(*tamanho == 0) {                                                        // se ta vazio
printf("\nREGISTRO VAZIO!\n\n");                                       // mostra mensagem
ultimo_cod = 0;                                                        // reseta o ultimo codigo
return;                                                                // sai da funcao
}
int posicao, i, codigo;                                                    // variaveis
char confirma = 'n';                                                       // variavel de confirmacao
printf("Codigo a ser excluido......:");                                    // pede o codigo
scanf("%d", &codigo);                                                      // le o codigo
limpar_buffer();                                                           // limpa o buffer
posicao = pesquisabinaria(estoque, codigo, *tamanho);                      // busca o produto
if (posicao >= 0) {                                                        // se achou o produto

printCabecalho("Produto para Exclus�o");                               // imprime titulo
printf(COR_VERDE "C�digo: %-15ld" COR_RESET, estoque[posicao].codigo); // mostra codigo
printf("%*s", 45, "");                                                 // espacamento
printf(COR_VERDE "Grupo: %d\n" COR_RESET, estoque[posicao].grupo);     // mostra grupo

printf(COR_DADOS "Descri��o: %-40s" COR_RESET, estoque[posicao].descricao); // mostra descricao
printf("%*s", 17, "");                                                 // espacamento
printf(COR_CIANO "Unidade: %s\n" COR_RESET, estoque[posicao].unidade); // mostra unidade

printf(COR_AMARELO "\nConfirma a exclusao do registro desse produto? (S/N)\n\n" COR_RESET); // pede confirmacao
confirma = getchar();                                                  // le S ou N
limpar_buffer();                                                       // limpa o buffer
if (confirma == 's' || confirma == 'S') {                              // se confirmou
if(estoque[posicao].codigo == ultimo_cod) {                        // se � o ultimo codigo
ultimo_cod = 0;                                                // reseta
for(i = 0; i < *tamanho; i++) {                                // procura o novo ultimo codigo
if(i != posicao && estoque[i].codigo > ultimo_cod) ultimo_cod = estoque[i].codigo; // pega o maior
}
}
// remove "puxando" todo mundo pra tras (shift left)
for (i = posicao; i < (*tamanho) - 1; i++) estoque[i] = estoque[i + 1]; // copia o proximo pro lugar atual
(*tamanho)--;                                                      // diminui o tamanho
if(*tamanho == 0) ultimo_cod = 0;                                  // se ficou vazio reseta o codigo
printSucesso("REGISTRO REMOVIDO!\n\n");                            // mostra mensagem de sucesso
} else printf("\n O REGRISTRO NAO FOI EXCLUIDO!\n\n");                 // se nao confirmou
} else printErro("O REGRISTRO NAO FOI LOCALIZADO!\n\n");                   // se nao achou
printf("APERTE ENTER PARA VOLTAR AO MENU");                                // pede pra voltar
getchar();                                                                 // espera
system("cls");                                                             // limpa a tela
}

//============================================================================
// mostra alerta de estoque baixo (quando ta abaixo do minimo)
void alerta_estoque_min(TProduto estoque[], int tamanho) {             // funcao de alerta
int encontrou = 0;                                                    // flag pra saber se achou algum

if (tamanho == 0) {                                                     // se nao tem produtos
printf("\nNenhum produto cadastrado.\n");                              // mostra mensagem
printf("Pressione ENTER para continuar...");                           // pede pra continuar
limpar_buffer();                                                       // limpa o buffer
getchar();                                                             // espera
return;                                                                // sai da funcao
}

printCabecalho("Produtos com Estoque Abaixo do M�nimo");                  // imprime titulo

for (int i = 0; i < tamanho; i++) {                                        // percorre todos os produtos
if (estoque[i].quantidade <= estoque[i].estoque_min) {                 // se ta abaixo do minimo

printf(COR_VERMELHO "C�digo: %ld   Descri��o: %s\n" COR_RESET,     // mostra codigo e descricao em vermelho
estoque[i].codigo, estoque[i].descricao);

printf(COR_AMARELO "Quantidade: %.2f   Estoque M�nimo: %.2f\n" COR_RESET, // mostra quantidade e minimo
estoque[i].quantidade, estoque[i].estoque_min);

printf(COR_DADOS "Fornecedor: %s   Pre�o Venda: %.2f\n" COR_RESET, // mostra fornecedor e preco
estoque[i].fornecedor, estoque[i].pr_venda);

printLinha();                                                      // imprime linha divisoria
encontrou = 1;                                                     // marca que encontrou pelo menos um
}
}

if (!encontrou) {                                                          // se nao encontrou nenhum
printf("\nNenhum produto com estoque abaixo do m�nimo.\n");            // mostra mensagem
}

printf("\nPressione ENTER para continuar...");                             // pede pra continuar
limpar_buffer();                                                           // limpa o buffer
getchar();                                                                 // espera
}



void fracionar_produto(TProduto estoque[], int *tamanho) {
if (*tamanho == 0) {                                // nenhum produto cadastrado
printf("\nNenhum produto cadastrado.\n");
printf("Pressione ENTER para continuar...");
limpar_buffer();                                // limpa buffer
getchar();                                      // pausa
return;
}

long int codigo_busca;                              // c�digo a buscar
float qtd_fracionar;                                // quantidade a fracionar

printf("\nC�digo do produto a fracionar: ");
scanf("%ld", &codigo_busca);                        // recebe c�digo
limpar_buffer();                                    // limpa buffer

int pos = -1;                                       // posi��o do produto
for (int i = 0; i < *tamanho; i++) {                // percorre estoque
if (estoque[i].codigo == codigo_busca) {        // encontrou?
pos = i;
break;                                      // para o loop
}
}

if (pos == -1) {                                    // n�o achou
printf("\nProduto n�o encontrado.\n");
printf("Pressione ENTER para continuar...");
getchar();
return;
}

printf("\nQuantidade a fracionar (ex: 0.250, 1.0, 2.5 kg): ");
scanf("%f", &qtd_fracionar);                        // recebe quantidade
limpar_buffer();                                    // limpa buffer

if (qtd_fracionar <= 0) {                           // quantidade inv�lida
printf("\nQuantidade inv�lida.\n");
printf("Pressione ENTER para continuar...");
getchar();
return;
}

if (estoque[pos].quantidade < qtd_fracionar) {      // checa se tem estoque suficiente
printf("\nQuantidade insuficiente no estoque.\n");
printf("Dispon�vel: %.3f\n", estoque[pos].quantidade);
printf("Pressione ENTER para continuar...");
getchar();
return;
}

estoque[pos].quantidade -= qtd_fracionar;           // desconta do produto original

TProduto novo;                                      // novo item fracionado
novo.codigo = ++ultimo_cod;                         // gera novo c�digo

novo.grupo = estoque[pos].grupo;                    // copia grupo
snprintf(novo.descricao, 40, "%s (Fracionado)", 
estoque[pos].descricao);                   // adiciona "(Fracionado)" na descri��o

strcpy(novo.unidade, estoque[pos].unidade);         // copia unidade
strcpy(novo.fornecedor, estoque[pos].fornecedor);   // copia fornecedor

novo.quantidade = qtd_fracionar;                    // define quantidade fracionada
novo.pr_compra = estoque[pos].pr_compra;            // copia pre�o de compra
novo.pr_venda = estoque[pos].pr_venda;              // copia pre�o de venda
novo.lucro = estoque[pos].lucro;                    // copia lucro
novo.estoque_min = 0;                               // item fracionado n�o tem m�nimo

estoque[*tamanho] = novo;                           // adiciona ao vetor
(*tamanho)++;                                       // incrementa tamanho

printSucesso("\nProduto fracionado criado com sucesso!");  // mensagem de sucesso
printf("C�digo novo: %ld\n", novo.codigo);          // exibe c�digo
printf("Descri��o: %s\n", novo.descricao);          // exibe descri��o
printf("Quantidade: %.3f\n", novo.quantidade);       // exibe quantidade

printf("\nPressione ENTER para continuar...");
getchar();                                          // pausa
}



void printRel(TProduto estoque[], int posicao) {  // recebe vetor E posi��o
printf(COR_CIANO "C�digo:" COR_BRANCO " %-30ld" COR_RESET, estoque[posicao].codigo);        // c�digo
printf("%*s", 45, "");                                                                     // espa�amento
printf(COR_CIANO "Grupo:" COR_BRANCO " %d\n" COR_RESET, estoque[posicao].grupo);             // grupo

printf(COR_CIANO "Descri��o:" COR_BRANCO " %-62s" COR_RESET, estoque[posicao].descricao);    // descri��o
printf("%*s", 10, "");                                                                      // espa�amento
printf(COR_CIANO "Unidade:" COR_BRANCO " %s\n" COR_RESET, estoque[posicao].unidade);         // unidade

printf(COR_CIANO "Fornecedor:" COR_BRANCO " %s\n" COR_RESET, estoque[posicao].fornecedor);   // fornecedor

printf(COR_CIANO "Pre�o de Compra:" COR_BRANCO " %-20.2f" COR_RESET, estoque[posicao].pr_compra);  // compra
printf("%*s", 5, "");                                                                       // espa�amento

printf(COR_CIANO "Pre�o de Venda:" COR_BRANCO " %-14.2f" COR_RESET, estoque[posicao].pr_venda);     // venda
printf("%*s", 5, "");                                                                       // espa�amento

printf(COR_CIANO "Lucro M�nimo:" COR_BRANCO " %d%%\n" COR_RESET, estoque[posicao].lucro);    // lucro

printf(COR_CIANO "Quantidade:" COR_BRANCO " %-33.0f" COR_RESET, estoque[posicao].quantidade);       // quantidade
printf("%*s", 25, "");                                                                      // espa�amento

printf(COR_CIANO "Quantidade M�nima:" COR_BRANCO " %.2f\n" COR_RESET, estoque[posicao].estoque_min); // estoque min
}
