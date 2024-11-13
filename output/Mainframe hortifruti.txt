#include <stdlib.h>
#include <ctype.h>
#include <stdio.h>
#include <string.h>
#include <time.h>       //necessária apenas para criar uma chave única na etiqueta

#define ETIQUETAS "etiquetas.dat"
#define ESTOQUE "estoque.dat"

// Estrutura para representar um item (produto) no estoque
typedef struct {
    int id;
    char nome[100];
    float quantidade;
    float preco;
} Item;
// Estrutura para representar uma etiqueta para faturamento no caixa
typedef struct {
    long idEtiqueta;
    int idProduto;
    char nomeProduto[100];
    float peso;
    float precoFinal;
    int faturada;
    char dataFaturamento[20];
} Etiqueta;
void cadastrarItem() {
    FILE *file = fopen(ESTOQUE, "r+b"); // Abrir no modo leitura/escrita binário
    Item item;
    int id;
    char entrada[10]; // Buffer para captura de entrada do ID e kg
    char *endptr;

    if (!file) {
        printf("Erro ao abrir o arquivo!\n");
        return;
    }

    // Captura e valida o ID do produto
    printf("\n========Cadastro de produtos========\n\n");
    do {
        printf("ID: ");
        fgets(entrada, sizeof(entrada), stdin);

        id = strtol(entrada, &endptr, 10);
        if (*endptr != '\n' || id <= 0) {
            printf("ID invalido!\n");
            continue;
        }

        // Verifica se o ID já existe no arquivo
        rewind(file); // Rebobina para o início para verificar duplicados
        int idExistente = 0;
        while (fread(&item, sizeof(Item), 1, file)) {
            if (item.id == id) {
                printf("Produto ja cadastrado com esse ID. Excluir ou alterar o registro atual.\n");
                idExistente = 1;
                break;
            }
        }
        if (idExistente) continue;

        // Se o ID é único, associamos ao item
        item.id = id;
        break;

    } while (1);

    // Solicita e captura o nome do produto
    printf("Nome: ");
    fgets(item.nome, sizeof(item.nome), stdin);
    size_t len = strlen(item.nome);
    if (len > 0 && item.nome[len - 1] == '\n') {
        item.nome[len - 1] = '\0'; // Remove a nova linha
    }

    // Captura e valida o valor em kg
    do {
        printf("Volume inicial em estoque (Kg): ");
        fgets(entrada, sizeof(entrada), stdin);

        item.quantidade = strtof(entrada, &endptr);
        if (*endptr != '\n' || item.quantidade <= 0) {
            printf("Volume invalido!\n");
            continue;
        }
        break;
    } while (1);

    // Captura e valida o preço
    do {
        printf("Valor por KG: ");
        fgets(entrada, sizeof(entrada), stdin);

        item.preco = strtof(entrada, &endptr);
        if (*endptr != '\n' || item.preco <= 0) {
            printf("Valor invalido! .\n");
            continue;
        }
        break;
    } while (1);

    // Rebobina o arquivo para o final e grava o novo item
    fseek(file, 0, SEEK_END);
    fwrite(&item, sizeof(Item), 1, file);
    fclose(file);

    printf("Produto cadastrado com sucesso!\n");
    system("pause");
}
void alterarItem() {
    FILE *file = fopen(ESTOQUE, "r+b");
    Item item;
    int id, alteracao, encontrado = 0;

    if (!file) {
        printf("Erro ao abrir o arquivo!\n");
        return;
    }

    printf("ID do item a ser alterado: ");
    scanf("%d", &id);

    // Lê o arquivo e procura o item pelo ID
    while (fread(&item, sizeof(Item), 1, file)) {
        if (item.id == id) {
            printf("\nNome atual: %s\n", item.nome);
            printf("Valor atual: %.2f\n", item.preco);
            printf("Volume atual: %.2f\n", item.quantidade);

            do {
                // Menu de opções de alteração
                printf(
                    "\n1 - Alterar nome"
                    "\n2 - Alterar valor por KG"
                    "\n0 - Voltar"
                    "\nEscolha uma opcao: ");
                
                scanf("%d", &alteracao);

                switch (alteracao) {
                    case 1:
                        printf("\nNovo Nome: ");
                        fgets(item.nome, sizeof(item),stdin);
                        break;
                    case 2:
                        printf("\nNovo valor: ");
                        scanf("%f", &item.preco);
                        break;
                    case 0:
                        printf("Voltando ao menu principal...\n");
                        break;
                    default:
                        printf("Opcao invalida!\n");
                        break;
                }

            } while (alteracao != 0); // Sai do loop se o usuário escolher voltar (0)

            // Move o ponteiro de volta para sobrescrever o item
            fseek(file, -sizeof(Item), SEEK_CUR);
            fwrite(&item, sizeof(Item), 1, file);

            encontrado = 1;
            break; // Encerra a busca, pois o item já foi encontrado e alterado
        }
    }

    if (!encontrado) {
        printf("Item nao encontrado!\n");
    }

    fclose(file);
    system("pause");
}
void excluirItem() {
    FILE *file = fopen(ESTOQUE, "rb");
    FILE *tempFile = fopen("temp.dat", "wb");
    Item item;
    int id, encontrado = 0;
    char confirmacao[4];

    if (!file || !tempFile) {
        printf("Erro ao abrir o arquivo!\n");
        return;
    }

    printf("ID do item a ser excluido: ");
    scanf("%d", &id);

    while (fread(&item, sizeof(Item), 1, file)) {
        if (item.id == id) {
            encontrado = 1;
            printf("\nItem encontrado:\n");
            printf("Nome: %s\n", item.nome);
            printf("Preco: %.2f\n", item.preco);
            printf("Volume atual: %.2f Kg\n", item.quantidade);

            printf("\nTem certeza que deseja excluir este item? Digite 'SIM' para confirmar: ");
            scanf("%s", confirmacao);

            // Verifica se a confirmação é exatamente "sim", "Sim" ou "SIM"
            if (strcmp(confirmacao, "SIM") == 0 || strcmp(confirmacao, "Sim") == 0 || strcmp(confirmacao, "sim") == 0) {
                printf("Item excluido!\n");
            } else {
                // Caso a confirmação não seja válida, gravar o item de volta no arquivo temporário
                fwrite(&item, sizeof(Item), 1, tempFile);
                printf("Exclusao cancelada!\n");
                encontrado = 0;  // Marca como não excluído para não exibir mensagem de sucesso
            }
        } else {
            // Escreve os itens que não foram excluídos no arquivo temporário
            fwrite(&item, sizeof(Item), 1, tempFile);
        }
    }

    fclose(file);
    fclose(tempFile);

    // Substitui o arquivo original pelo arquivo temporário
    remove(ESTOQUE);
    rename("temp.dat", ESTOQUE);

    if (!encontrado) {
        printf("Item nao encontrado ou exclusao cancelada!\n");
    }
        system("pause");
}
void listarItens() {
    FILE *file = fopen(ESTOQUE, "rb");
    Item item;

    if (!file) {
        printf("Erro ao abrir o arquivo!\n");
        return;
    }

    printf("\nItens no estoque:\n");

    while (fread(&item, sizeof(Item), 1, file)) {
        printf("============================\n");
        printf("ID: %d\n", item.id);
        printf("Nome: %s\n", item.nome);
        printf("Volume: %.2f Kg\n", item.quantidade);
        printf("Preco: %.2f\n", item.preco);
        printf("============================\n\n");
    }

    fclose(file);
    system("pause");
}
void entradaEstoque() {
    FILE *file = fopen(ESTOQUE, "r+b");  // Abrir o arquivo no modo leitura/escrita binário
    Item item;
    int id,  encontrado = 0;
    float quantidadeEntrada;

    if (!file) {
        printf("Erro ao abrir o arquivo!\n");
        return;
    }

    printf("ID do item recebido: ");
    scanf("%d", &id);

    // Procura o item no arquivo pelo ID
    while (fread(&item, sizeof(Item), 1, file)) {
        if (item.id == id) {
            printf("Item encontrado: %s\n", item.nome);
            printf("Volume atual: %.2f Kg\n", item.quantidade);

            // Solicita a quantidade de entrada
            printf("Quantidade recebida: ");
            scanf("%f", &quantidadeEntrada);

            // Atualiza a quantidade do item
            item.quantidade += quantidadeEntrada;
            printf("Novo volume: %.2f\n", item.quantidade);

            // Mover o ponteiro de arquivo para a posição anterior (para reescrever o registro atualizado)
            fseek(file, -sizeof(Item), SEEK_CUR);

            // Reescreve o item atualizado
            fwrite(&item, sizeof(Item), 1, file);
            encontrado = 1;
            break;
        }
    }

    if (!encontrado) {
        printf("Item nao encontrado!\n");
    }

    fclose(file);
    system("pause");
}
void consultarItem() {
    FILE *file = fopen(ESTOQUE, "rb");
    Item item;
    int id, encontrado = 0;

    if (!file) {
        printf("Erro ao abrir o arquivo!\n");
        return;
    }

    printf("Digite o ID do produto a ser consultado: ");
    scanf("%d", &id);

    // Lê o arquivo e procura o item pelo ID
    while (fread(&item, sizeof(Item), 1, file)) {
        if (item.id == id) {
            printf("\n-- Produto Encontrado --\n");
            printf("ID: %d\n", item.id);
            printf("Nome: %s\n", item.nome);
            printf("Volume: %.2f\n", item.quantidade);
            printf("Preco: %.2f\n", item.preco);
            printf("------------------------\n");
            encontrado = 1;
            break; // Encerra a busca após encontrar o item
        }
    }

    if (!encontrado) {
        printf("Produto com ID %d nao encontrado!\n", id);
    }

    fclose(file);
    system("pause");
}
void balanca() {
    FILE *file = fopen(ESTOQUE, "rb");
    FILE *fileEtiquetas = fopen(ETIQUETAS, "ab");  // Abre o arquivo de etiquetas em modo de anexo
    Item item;
    int id, encontrado = 0;
    float peso, precoFinal;
    long etiquetaID;
    time_t t = time(NULL);  // Usar timestamp para gerar uma etiqueta única

    if (!file || !fileEtiquetas) {
        printf("Erro ao abrir o arquivo!\n");
        return;
    }

    printf("Digite o ID do produto a ser pesado: ");
    scanf("%d", &id);

    // Lê o arquivo e procura o item pelo ID
    while (fread(&item, sizeof(Item), 1, file)) {
        if (item.id == id) {
            printf("Produto encontrado: %s\n", item.nome);

            printf("Digite o peso em gramas: ");
            scanf("%f", &peso);

            precoFinal = (peso / 1000) * item.preco;

            // Gera o número da etiqueta (timestamp + ID)
            etiquetaID = t + id;

            printf("\n--- Etiqueta ---\n");
            printf("Numero da Etiqueta: %ld\n", etiquetaID);
            printf("Produto: %s\n", item.nome);
            printf("Peso: %.2f g\n", peso);
            printf("Preco por Kg: R$ %.2f\n", item.preco);
            printf("Preco final: R$ %.2f\n", precoFinal);
            printf("----------------\n");

            // Armazena a etiqueta no arquivo
            Etiqueta etiqueta;
            etiqueta.idEtiqueta = etiquetaID;
            etiqueta.idProduto = id;
            etiqueta.peso = peso;
            etiqueta.precoFinal = precoFinal;
            etiqueta.faturada = 0;  // Não faturada inicialmente
            strcpy(etiqueta.nomeProduto, item.nome);

            fwrite(&etiqueta, sizeof(Etiqueta), 1, fileEtiquetas);

            encontrado = 1;
            break;
        }
    }

    if (!encontrado) {
        printf("Produto com ID %d nao encontrado!\n", id);
    }

    fclose(file);
    fclose(fileEtiquetas);
    system("pause");
}
void listarEtiquetas() {
    FILE *file = fopen(ETIQUETAS, "rb");
    Etiqueta etiqueta;

    if (!file) {
        printf("Erro ao abrir o arquivo!\n");
        return;
    }

    printf("\nEtiquetas nao faturadas:\n");

    while (fread(&etiqueta, sizeof(Etiqueta), 1, file)) {
        // if (etiqueta.faturada == 1) {
            printf("ID da Etiqueta: %ld\n", etiqueta.idEtiqueta);
            printf("ID do Produto: %d\n", etiqueta.idProduto);
            printf("Nome do Produto: %s\n", etiqueta.nomeProduto);
            printf("Volume: %.2f kg\n", etiqueta.peso);
            printf("Preco Final: R$ %.2f\n", etiqueta.precoFinal);
                if(etiqueta.faturada == 1){
            printf("Status de faturamento: Faturada\n");
            printf("Data de faturamento: %s\n", etiqueta.dataFaturamento);

            } else {
            printf("Status de faturamento: Nao faturada\n");
            }
            printf("============================\n\n");
        // }
    }

    fclose(file);
    system("pause");
}
void faturamentoEtiquetas() {
    FILE *fileEtiquetas = fopen(ETIQUETAS, "rb+");
    FILE *fileEstoque = fopen(ESTOQUE, "r+b");
    Etiqueta etiqueta;
    Item item;
    long idEtiqueta;
    int encontrado = 0;

    if (!fileEtiquetas || !fileEstoque) {
        printf("Erro ao abrir os arquivos!\n");
        return;
    }

    printf("Digite o numero da etiqueta: ");
    scanf("%ld", &idEtiqueta);

    // Procura a etiqueta pelo ID
    while (fread(&etiqueta, sizeof(Etiqueta), 1, fileEtiquetas)) {
        if (etiqueta.idEtiqueta == idEtiqueta) {
            if (etiqueta.faturada == 1) {
                printf("Esta etiqueta ja foi faturada!\n");
                encontrado = 1;
                break;
            }

            printf("\n--- Informacoes da Etiqueta ---\n");
            printf("Produto: %s\n", etiqueta.nomeProduto);
            printf("Peso: %.2f g\n", etiqueta.peso);
            printf("Preco final: R$ %.2f\n", etiqueta.precoFinal);
            printf("-------------------------------\n");

            // Confirma a finalização da transação
            char confirmar[4];
            printf("Confirmar a finalizacao da transacao? (Sim/Nao): ");
            scanf("%s", confirmar);

            // Verifica se a confirmação é exatamente "sim", "Sim" ou "SIM"
            if (strcmp(confirmar, "SIM") == 0 || strcmp(confirmar, "Sim") == 0 || strcmp(confirmar, "sim") == 0) {
                // Atualiza o estoque
                while (fread(&item, sizeof(Item), 1, fileEstoque)) {
                    if (item.id == etiqueta.idProduto) {
                        item.quantidade -= (int)(etiqueta.peso / 1000);
                        fseek(fileEstoque, -sizeof(Item), SEEK_CUR);
                        fwrite(&item, sizeof(Item), 1, fileEstoque);
                        break;
                    }
                }

                // Captura a data e hora atuais
                time_t t = time(NULL);
                struct tm tm = *localtime(&t);
                strftime(etiqueta.dataFaturamento, sizeof(etiqueta.dataFaturamento), "%Y-%m-%d %H:%M:%S", &tm);

                // Marca a etiqueta como faturada
                etiqueta.faturada = 1;
                fseek(fileEtiquetas, -sizeof(Etiqueta), SEEK_CUR);
                fwrite(&etiqueta, sizeof(Etiqueta), 1, fileEtiquetas);

                printf("Transacao finalizada e estoque atualizado!\n");
                printf("Data e hora de faturamento: %s\n", etiqueta.dataFaturamento);
            } else {
                printf("Transacao cancelada.\n");
            }

            encontrado = 1;
            break;
        }
    }

    if (!encontrado) {
        printf("Etiqueta com numero %ld nao encontrada!\n", idEtiqueta);
    }

    fclose(fileEtiquetas);
    fclose(fileEstoque);
    system("pause");
}
void excluirEtiquetas() {
    FILE *file = fopen(ETIQUETAS, "rb");
    FILE *tempFile = fopen("temp.dat", "wb");
    Etiqueta etiqueta;
    char confirmacao[4];
    int encontrouNaoFaturada = 0;

    if (!file || !tempFile) {
        printf("Erro ao abrir o arquivo!\n");
        return;
    }

    printf("Deseja excluir todas as etiquetas nao faturadas? Digite 'SIM' para confirmar: ");
    scanf("%3s", confirmacao);  // Limita a leitura a 3 caracteres

    if (strcmp(confirmacao, "SIM") != 0 && strcmp(confirmacao, "Sim") != 0 && strcmp(confirmacao, "sim") != 0) {
        printf("Exclusao cancelada pelo usuario.\n");
        fclose(file);
        fclose(tempFile);
        remove("temp.dat");
        return;
    }

    while (fread(&etiqueta, sizeof(Etiqueta), 1, file)) {
        if (etiqueta.faturada == 1) {
            fwrite(&etiqueta, sizeof(Etiqueta), 1, tempFile);
        } else {
            encontrouNaoFaturada = 1;
        }
    }

    fclose(file);
    fclose(tempFile);

    if (remove(ETIQUETAS) != 0 || rename("temp.dat", ETIQUETAS) != 0) {
        printf("Erro ao substituir o arquivo de etiquetas.\n");
        return;
    }

    if (encontrouNaoFaturada) {
        printf("Todas as etiquetas nao faturadas foram excluidas com sucesso.\n");
    } else {
        printf("Nao havia etiquetas nao faturadas para excluir.\n");
    }

    system("pause");
}
void menuConsultas(){
    int opcao = -1;  // Inicializa fora do intervalo de opções válidas
    char entrada[10];  // Buffer para capturar a entrada do usuário
    char *endptr;      // Ponteiro para verificar a validade da conversão

    do {
        printf("\n======== Painel de consultas - rede VERDE - VIVA ========\n");
        printf("1. Consultar produto\n");
        printf("2. Visualizar estoque\n");
        printf("3. Listar etiquetas emitidas\n");
        printf("0. Voltar\n\n");
        printf("Escolha uma opcao: ");
        
        fgets(entrada, sizeof(entrada), stdin);  // Lê a entrada como string
        
        // Remove o caractere de nova linha, se presente
        entrada[strcspn(entrada, "\n")] = '\0';

        // Converte a string para número inteiro e verifica se é válida
        opcao = strtol(entrada, &endptr, 10);
        
        // Verifica se houve algum caractere inválido na conversão
        if (*endptr != '\0' || strlen(entrada) == 0) {
            printf("Opcao invalida! Por favor, insira um numero entre 0 e 3.\n");
            opcao = -1;  // Redefine `opcao` para manter o loop ativo
            continue;    // Volta ao início do loop
        }

        // Verifica se o valor está dentro do intervalo esperado
        if (opcao < 0 || opcao > 3) {
            printf("Opcao invalida! Por favor, insira um numero entre 0 e 3.\n");
            opcao = -1;  // Redefine `opcao` para manter o loop ativo
            continue;
        }

        switch (opcao) {
            case 1:
                consultarItem();
                break;
            case 2:
                listarItens();
                break;
            case 3:
                listarEtiquetas();
                break;
            case 0:
                printf("Voltando...\n");
                system("pause");
                break;
            default:
                printf("Opcao invalida! Por favor, insira um numero entre 0 e 5.\n");
        }
    } while (opcao != 0);
}
void controleEstoque(){
    int opcao = -1;  // Inicializa fora do intervalo de opções válidas
    char entrada[10];  // Buffer para capturar a entrada do usuário
    char *endptr;      // Ponteiro para verificar a validade da conversão

    do {
        printf("\n======== Controle de estoque - rede VERDE - VIVA ========\n");
        printf("1. Visualizar estoque\n");
        printf("2. Cadastrar produto\n");
        printf("3. Alterar cadastro de produto\n");
        printf("4. Excluir cadastro de produto\n");
        printf("5. Recebimento\n");
        printf("0. Voltar\n\n");
        printf("Escolha uma opcao: ");
        
        fgets(entrada, sizeof(entrada), stdin);  // Lê a entrada como string
        
        // Remove o caractere de nova linha, se presente
        entrada[strcspn(entrada, "\n")] = '\0';

        // Converte a string para número inteiro e verifica se é válida
        opcao = strtol(entrada, &endptr, 10);
        
        // Verifica se houve algum caractere inválido na conversão
        if (*endptr != '\0' || strlen(entrada) == 0) {
            printf("Opcao invalida! Por favor, insira um numero entre 0 e 5.\n");
            opcao = -1;  // Redefine `opcao` para manter o loop ativo
            continue;    // Volta ao início do loop
        }

        // Verifica se o valor está dentro do intervalo esperado
        if (opcao < 0 || opcao > 5) {
            printf("Opcao invalida! Por favor, insira um numero entre 0 e 5.\n");
            opcao = -1;  // Redefine `opcao` para manter o loop ativo
            continue;
        }

        switch (opcao) {
            case 1:
                listarItens();
                break;
            case 2:
                cadastrarItem();
                break;
            case 3:
                alterarItem();
                break;
            case 4:
                excluirItem();
                break;
            case 5:
                entradaEstoque();
                break;                                
            case 0:
                printf("Voltando...\n");
                system("pause");
                break;
            default:
                printf("Opcao invalida! Por favor, insira um numero entre 0 e 5.\n");
        }
    } while (opcao != 0);
}
void caixaRegistradora(){
    int opcao = -1;  // Inicializa fora do intervalo de opções válidas
    char entrada[10];  // Buffer para capturar a entrada do usuário
    char *endptr;      // Ponteiro para verificar a validade da conversão

    do {
        printf("\n======== Caixa registradora - rede VERDE - VIVA ========\n");
        printf("1. Faturar etiqueta\n");
        printf("2. Listar etiquetas emitidas\n");
        printf("3. Excluir etiquetas nao faturadas\n");
        printf("0. Voltar\n\n");
        printf("Escolha uma opcao: ");
        
        fgets(entrada, sizeof(entrada), stdin);  // Lê a entrada como string
        
        // Remove o caractere de nova linha, se presente
        entrada[strcspn(entrada, "\n")] = '\0';

        // Converte a string para número inteiro e verifica se é válida
        opcao = strtol(entrada, &endptr, 10);
        
        // Verifica se houve algum caractere inválido na conversão
        if (*endptr != '\0' || strlen(entrada) == 0) {
            printf("Opcao invalida! Por favor, insira um numero entre 0 e 3.\n");
            opcao = -1;  // Redefine `opcao` para manter o loop ativo
            continue;    // Volta ao início do loop
        }

        // Verifica se o valor está dentro do intervalo esperado
        if (opcao < 0 || opcao > 3) {
            printf("Opcao invalida! Por favor, insira um numero entre 0 e 3.\n");
            opcao = -1;  // Redefine `opcao` para manter o loop ativo
            continue;
        }

        switch (opcao) {
            case 1:
                faturamentoEtiquetas();
                break;
            case 2:
                listarEtiquetas();
                break;
            case 3:
                excluirEtiquetas();
                break;
            case 0:
                printf("Voltando...\n");
                system("pause");
                break;
            default:
                printf("Opcao invalida! Por favor, insira um numero entre 0 e 5.\n");
        }
    } while (opcao != 0);
}
// Iniciação do programa
int main() {
    int opcao = -1;  // Inicializa fora do intervalo de opções válidas
    char entrada[10];  // Buffer para capturar a entrada do usuário
    char *endptr;      // Ponteiro para verificar a validade da conversão

    do {
        printf("\n======== Mainframe - Rede VERDE-VIVA ========\n");
        printf("1. Consultas\n");
        printf("2. Controle de estoque\n");
        printf("3. Caixa registradora\n");
        printf("4. Pesagem para venda\n");
        printf("0. Sair\n\n");
        printf("Escolha uma opcao: ");
        
        fgets(entrada, sizeof(entrada), stdin);  // Lê a entrada como string
        
        // Remove o caractere de nova linha, se presente
        entrada[strcspn(entrada, "\n")] = '\0';

        // Converte a string para número inteiro e verifica se é válida
        opcao = strtol(entrada, &endptr, 10);
        
        // Verifica se houve algum caractere inválido na conversão
        if (*endptr != '\0' || strlen(entrada) == 0) {
            printf("Opcao invalida! Por favor, insira um numero entre 0 e 4.\n");
            opcao = -1;  // Redefine `opcao` para manter o loop ativo
            continue;    // Volta ao início do loop
        }

        // Verifica se o valor está dentro do intervalo esperado
        if (opcao < 0 || opcao > 4) {
            printf("Opcao invalida! Por favor, insira um numero entre 0 e 4.\n");
            opcao = -1;  // Redefine `opcao` para manter o loop ativo
            continue;
        }

        switch (opcao) {
            case 1:
                menuConsultas();
                break;
            case 2:
                controleEstoque();
                break;
            case 3:
                caixaRegistradora();
                break;
            case 4:
                balanca();
                break;
            case 0:
                printf("Saindo...\n");
                break;
            default:
                printf("Opcao invalida! Por favor, insira um numero entre 0 e 4.\n");
        }
    } while (opcao != 0);

    return 0;
}
