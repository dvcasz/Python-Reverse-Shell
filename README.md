# Python Reverse Shell

Este projeto contém um script em Python que funciona como um **reverse shell** (ou backdoor). Ele foi criado para fins estritamente **educacionais** 🎓, com o objetivo de demonstrar como malwares básicos operam para estabelecer persistência e comunicação com um servidor de Comando e Controle (C2).

**⚠️ AVISO IMPORTANTE**
Este código permite a execução remota de comandos. Utilizá-lo em qualquer sistema sem a permissão explícita do proprietário é ilegal e antiético. O autor não se responsabiliza por qualquer mau uso deste script.

---

## O que ele faz?

O script, ao ser executado em uma máquina-alvo (Windows), realiza as seguintes ações:

1.  **Persistência**: Copia a si mesmo (em formato `.exe`) para a pasta de inicialização do Windows (`Startup`), garantindo que o programa seja executado toda vez que o usuário fizer login.
2.  **Conexão Reversa**: Tenta se conectar a um endereço de IP e porta pré-definidos (o servidor de Comando e Controle). A conexão é "reversa" porque parte da máquina infectada para o atacante, uma técnica comum para contornar firewalls.
3.  **Loop de Reconexão**: Se a conexão com o servidor C2 falhar ou cair, o script aguarda 5 segundos e tenta se reconectar infinitamente.
4.  **Execução Remota de Comandos**: Uma vez conectado, o script aguarda por comandos enviados pelo servidor C2. Qualquer comando recebido é executado diretamente no `shell` do sistema operacional da vítima.
5.  **Retorno do Resultado**: A saída (resultado) do comando executado é enviada de volta para o servidor C2, permitindo que o operador veja o que aconteceu.

---

## Análise do Código

O script é dividido nas seguintes funções:

### `autorun()`
* **Objetivo**: Estabelecer persistência no sistema.
* **Como**: Ele identifica o nome do arquivo executável e o copia para o diretório `%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup`.

### `conn(CCIP, CCPORT)`
* **Objetivo**: Conectar-se ao servidor de Comando e Controle (C2).
* **Como**: Cria um socket TCP e tenta estabelecer uma conexão com o IP e a porta definidos nas variáveis globais `CCIP` e `CCPORT`.

### `cmd(client, data)`
* **Objetivo**: Executar o comando recebido do C2.
* **Como**: Utiliza o módulo `subprocess.Popen` com `shell=True` para executar a string de comando (`data`). A saída padrão (`stdout`) e os erros (`stderr`) são capturados e enviados de volta ao C2 através da conexão do socket.

### `cli(client)`
* **Objetivo**: Manter a comunicação com o C2, recebendo comandos.
* **Como**: Entra em um loop infinito, aguardando dados (`recv`) do servidor. Cada comando recebido é processado em uma nova `thread`, permitindo que o backdoor lide com múltiplas tarefas sem travar. Possui um comando especial, `/:kill`, para encerrar a conexão.

---

## Como Usar (em Ambiente Controlado)

Para testar este script, você precisará de duas máquinas: uma para o atacante (servidor C2) e uma para a vítima.

#### 1. No Servidor do Atacante (C2)

Abra um terminal e use uma ferramenta como o **Netcat** para escutar por conexões na porta definida no script (por padrão, a porta `443`).

```bash
# O 'ncat' ou 'nc' ficará aguardando a conexão do script
ncat -lvnp 443
```

#### 2. No Script (Cliente)
Antes de executar na máquina-alvo, você deve alterar a variável CCIP:

```bash
# Altere "0.0.0.0" para o endereço IP da sua máquina de atacante
CCIP = "SEU_IP_AQUI"
CCPORT = 443
```

#### 3. Na Máquina-Alvo
Execute o script Python. Se você o compilou para .exe (usando ferramentas como pyinstaller), execute o arquivo .exe. Assim que for executado, ele tentará se conectar ao seu listener Netcat.
Quando a conexão for estabelecida, você poderá digitar comandos de shell (como dir, whoami, ipconfig) diretamente no seu terminal Netcat, e a saída aparecerá na sua tela.
