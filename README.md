# tetris.c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define TAM_FILA 5
#define TAM_PILHA 3

// Definição da estrutura da peça
typedef struct {
    char nome;  // Tipo da peça: 'I', 'O', 'T', 'L'
    int id;     // Identificador único
} Peca;

// Fila circular de peças futuras
typedef struct {
    Peca fila[TAM_FILA];
    int inicio;
    int fim;
    int tamanho;
} FilaPecas;

// Pilha de peças reservadas
typedef struct {
    Peca pilha[TAM_PILHA];
    int topo;  // -1 significa pilha vazia
} PilhaReserva;

int contadorID = 0;  // Controla o id único das peças

// Gera uma nova peça aleatória com id único
Peca gerarPeca() {
    char tipos[] = {'I', 'O', 'T', 'L'};
    Peca p;
    p.nome = tipos[rand() % 4];
    p.id = contadorID++;
    return p;
}

// Inicializa a fila preenchendo com peças geradas
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

// Inicializa a pilha vazia
void inicializarPilha(PilhaReserva *p) {
    p->topo = -1;
}

// Remove peça da frente da fila (dequeue)
Peca removerDaFila(FilaPecas *f) {
    Peca p = {' ', -1};
    if (f->tamanho == 0) {
        printf("Fila vazia!\n");
        return p;
    }
    p = f->fila[f->inicio];
    f->inicio = (f->inicio + 1) % TAM_FILA;
    f->tamanho--;
    return p;
}

// Adiciona peça ao final da fila (enqueue)
void adicionarNaFila(FilaPecas *f, Peca p) {
    if (f->tamanho == TAM_FILA) {
        printf("Fila cheia!\n");
        return;
    }
    f->fila[f->fim] = p;
    f->fim = (f->fim + 1) % TAM_FILA;
    f->tamanho++;
}

// Empilha peça no topo da pilha
int empilhar(PilhaReserva *p, Peca peca) {
    if (p->topo == TAM_PILHA - 1) {
        printf("Pilha cheia! Não é possível reservar mais peças.\n");
        return 0;
    }
    p->topo++;
    p->pilha[p->topo] = peca;
    return 1;
}

// Remove peça do topo da pilha (pop)
Peca desempilhar(PilhaReserva *p) {
    Peca peca = {' ', -1};
    if (p->topo == -1) {
        printf("Pilha vazia! Nenhuma peça reservada.\n");
        return peca;
    }
    peca = p->pilha[p->topo];
    p->topo--;
    return peca;
}

// Troca a peça da frente da fila com o topo da pilha
void trocarTopoFilaComTopoPilha(FilaPecas *f, PilhaReserva *p) {
    if (f->tamanho == 0) {
        printf("Fila vazia, não há peça para trocar.\n");
        return;
    }
    if (p->topo == -1) {
        printf("Pilha vazia, não há peça para trocar.\n");
        return;
    }
    Peca temp = f->fila[f->inicio];
    f->fila[f->inicio] = p->pilha[p->topo];
    p->pilha[p->topo] = temp;
    printf("Troca realizada entre peça da frente da fila e topo da pilha.\n");
}

// Troca as 3 primeiras peças da fila com as 3 peças da pilha
void trocaTripla(FilaPecas *f, PilhaReserva *p) {
    if (f->tamanho < 3) {
        printf("Fila não tem 3 peças para troca.\n");
        return;
    }
    if (p->topo < 2) {
        printf("Pilha não tem 3 peças para troca.\n");
        return;
    }

    // Trocar as 3 primeiras peças da fila com as 3 peças da pilha
    for (int i = 0; i < 3; i++) {
        int idxFila = (f->inicio + i) % TAM_FILA;
        // Pilha tem topo em p->topo, e a base em 0. Vamos trocar com os 3 do topo para base (topo, topo-1, topo-2)
        Peca temp = f->fila[idxFila];
        f->fila[idxFila] = p->pilha[p->topo - i];
        p->pilha[p->topo - i] = temp;
    }
    printf("Troca múltipla realizada entre os 3 primeiros da fila e as 3 peças da pilha.\n");
}

// Exibe o estado da fila
void exibirFila(FilaPecas *f) {
    printf("Fila de peças\t");
    if (f->tamanho == 0) {
        printf("(vazia)");
    } else {
        int idx = f->inicio;
        for (int i = 0; i < f->tamanho; i++) {
            Peca p = f->fila[idx];
            printf("[%c %d] ", p.nome, p.id);
            idx = (idx + 1) % TAM_FILA;
        }
    }
    printf("\n");
}

// Exibe o estado da pilha
void exibirPilha(PilhaReserva *p) {
    printf("Pilha de reserva\t(Topo -> base): ");
    if (p->topo == -1) {
        printf("(vazia)");
    } else {
        for (int i = p->topo; i >= 0; i--) {
            Peca peca = p->pilha[i];
            printf("[%c %d] ", peca.nome, peca.id);
        }
    }
    printf("\n");
}

// Exibe o menu de opções
void exibirMenu() {
    printf("\nOpções disponíveis:\n");
    printf("1 - Jogar peça da frente da fila\n");
    printf("2 - Enviar peça da fila para a pilha de reserva\n");
    printf("3 - Usar peça da pilha de reserva\n");
    printf("4 - Trocar peça da frente da fila com o topo da pilha\n");
    printf("5 - Trocar os 3 primeiros da fila com as 3 peças da pilha\n");
    printf("0 - Sair\n");
    printf("Opção escolhida: ");
}

int main() {
    FilaPecas fila;
    PilhaReserva pilha;
    int opcao;

    srand(time(NULL));

    inicializarFila(&fila);
    inicializarPilha(&pilha);

    do {
        printf("\n=== Estado atual ===\n");
        exibirFila(&fila);
        exibirPilha(&pilha);

        exibirMenu();
        scanf("%d", &opcao);

        switch (opcao) {
            case 1: { // Jogar peça da fila
                if (fila.tamanho == 0) {
                    printf("Fila vazia, não há peça para jogar.\n");
                    break;
                }
                Peca jogada = removerDaFila(&fila);
                printf("Jogando peça [%c %d]\n", jogada.nome, jogada.id);
                // Após remoção, gerar nova peça para manter fila cheia
                adicionarNaFila(&fila, gerarPeca());
                break;
            }
            case 2: { // Reservar peça (mover da fila para pilha)
                if (fila.tamanho == 0) {
                    printf("Fila vazia, não há peça para reservar.\n");
                    break;
                }
                if (pilha.topo == TAM_PILHA - 1) {
                    printf("Pilha cheia, não é possível reservar mais peças.\n");
                    break;
                }
                Peca reservada = removerDaFila(&fila);
                if (empilhar(&pilha, reservada)) {
                    printf("Peça [%c %d] enviada para a pilha de reserva.\n", reservada.nome, reservada.id);
                    // Gerar nova peça para fila
                    adicionarNaFila(&fila, gerarPeca());
                }
                break;
            }
            case 3: { // Usar peça da pilha (remover do topo)
                Peca usada = desempilhar(&pilha);
                if (usada.id != -1) {
                    printf("Usando peça reservada [%c %d]\n", usada.nome, usada.id);
                }
                break;
            }
            case 4: { // Trocar peça da frente da fila com topo da pilha
                trocarTopoFilaComTopoPilha(&fila, &pilha);
                break;
            }
            case 5: { // Troca múltipla das 3 primeiras peças da fila com 3 da pilha
                trocaTripla(&fila, &pilha);
                break;
            }
            case 0:
                printf("Encerrando o programa. Até logo!\n");
                break;
            default:
                printf("Opção inválida, tente novamente.\n");
        }
    } while (opcao != 0);

    return 0;
}
