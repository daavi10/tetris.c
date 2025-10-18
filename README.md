# tetris.c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define TAM_FILA 5

// Estrutura que representa uma peça do Tetris
typedef struct {
    char nome;  // Tipo da peça ('I', 'O', 'T', 'L')
    int id;     // Identificador único
} Peca;

// Fila circular de peças
typedef struct {
    Peca fila[TAM_FILA];
    int inicio;     // Índice do início da fila
    int fim;        // Índice do fim da fila
    int tamanho;    // Quantidade atual de elementos na fila
} FilaPecas;

int contadorID = 0;  // Controla o ID único de cada peça

// Gera uma nova peça aleatória
Peca gerarPeca() {
    char tipos[] = {'I', 'O', 'T', 'L'};
    Peca nova;
    nova.nome = tipos[rand() % 4]; // Escolhe aleatoriamente entre os 4 tipos
    nova.id = contadorID++;        // Garante ID único
    return nova;
}

// Inicializa a fila com 5 peças
void inicializarFila(FilaPecas *f) {
    f->inicio = 0;
    f->fim = 0;
    f->tamanho = 0;

    for (int i = 0; i < TAM_FILA; i++) {
        f->fila[f->fim] = gerarPeca();
        f->fim = (f->fim + 1) % TAM_FILA;
        f->tamanho++;
    }
}

// Insere nova peça no fim da fila
void enqueue(FilaPecas *f) {
    if (f->tamanho == TAM_FILA) {
        printf("Fila cheia! Não é possível inserir nova peça.\n");
        return;
    }

    f->fila[f->fim] = gerarPeca();
    f->fim = (f->fim + 1) % TAM_FILA;
    f->tamanho++;
}

// Remove a peça do início da fila
void dequeue(FilaPecas *f) {
    if (f->tamanho == 0) {
        printf("Fila vazia! Nenhuma peça para jogar.\n");
        return;
    }

    Peca p = f->fila[f->inicio];
    printf("Jogando peça: [%c %d]\n", p.nome, p.id);
    f->inicio = (f->inicio + 1) % TAM_FILA;
    f->tamanho--;
}

// Exibe o estado atual da fila
void exibirFila(FilaPecas *f) {
    printf("\nFila de peças:\n");
    int idx = f->inicio;

    for (int i = 0; i < f->tamanho; i++) {
        Peca p = f->fila[idx];
        printf("[%c %d] ", p.nome, p.id);
        idx = (idx + 1) % TAM_FILA;
    }
    if (f->tamanho == 0) {
        printf("(vazia)");
    }
    printf("\n");
}

// Exibe o menu de opções
void exibirMenu() {
    printf("\nOpções de ação:\n");
    printf("1 - Jogar peça (dequeue)\n");
    printf("2 - Inserir nova peça (enqueue)\n");
    printf("0 - Sair\n");
    printf("Escolha: ");
}

int main() {
    FilaPecas fila;
    int opcao;

    srand(time(NULL)); // Semente para números aleatórios

    inicializarFila(&fila);

    do {
        exibirFila(&fila);
        exibirMenu();
        scanf("%d", &opcao);

        switch (opcao) {
            case 1:
                dequeue(&fila);
                break;
            case 2:
                enqueue(&fila);
                break;
            case 0:
                printf("Encerrando o jogo. Até mais!\n");
                break;
            default:
                printf("Opção inválida! Tente novamente.\n");
        }

    } while (opcao != 0);

    return 0;
}
