# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: __9___ syscalls
- ex1b_write: ___7__ syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**

```
[ A diferença entre os dois metodos está na forma como o texto é enviado ao kernel. Quando é usado o printf o texto é armazenado no buffer durante a execução do programa e enviado pro kernel de uma vez quando o programa termina. Já o write não usa um buffer, toda vez que ele é chamado o programa faz uma conexão direta com o kernel para que o sistema operacional execute a operação.]

```

**3. Qual método é mais previsível? Por quê você acha isso?**

```
[O metodo mais previsivel é o Write, pois para cada chamada realisada é feita uma conexão direta como o kornel, garantindo a previsibilidade.]
```

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: ___3__
- Bytes lidos: __127___

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

```
[O número usado foi o 3. O sistema já usa os números 0, 1 e 2 para a entrada, a saída e os erros. Por isso, ele pegou o próximo número livre, que era o 3.

2. Como você sabe que o arquivo foi lido completamente?]
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
[Para confirmar que o arquivo foi lido, a função read retorna 0. Isso mostra o fim do arquivo. Como a função retornou 127, você só sabe que ela leu uma parte do arquivo.]
```

**3. Por que verificar retorno de cada syscall?**

```
[Verificar o retorno de cada função importante para evitar problemas. Se você não checar, seu programa pode tentar usar um arquivo que não foi lido corretamente]
```

---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: __25___ (esperado: 25)
- Caracteres: __1300___
- Chamadas read(): __21___
- Tempo: __61.9___ segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |       82        |   0.000182|
| 64          |       21        |   0.000085|
| 256         |       6         |   0.000056|
| 1024        |       2         |   0.000053|

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
[Quanto maior o buffer, menor a quantidade de vezes que o programa precisa chamar a função read para ler todo o arquivo.]
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

```
[Não, a última chamada de leitura sempre retorna menos bytes que o tamanho do buffer, porque ela só precisa pegar o que sobrou do arquivo.]
```

**3. Qual é a relação entre syscalls e performance?**

```
[Quanto menos chamadas de sistema melhor é performance. Cada vez que o seu programa faz uma chamada, ele tem que passar do "modo de usuário" para o "modo do sistema", o que gasta tempo. ]
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: __1364___
- Operações: ___6__
- Tempo: __0.000180___ segundos
- Throughput: __7400.17___ KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [x] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
[É para garantir que a cópia deu certo. Se o número de bytes escritos for diferente do que foi lido, algo deu errado e a cópia ficou incompleta.]
```

**2. Que flags são essenciais no open() do destino?**

```
[ Para o arquivo de destino, as flags essenciais são a O_WRONLY serve para abrir o arquivo só pra escrita. A O_CREAT cria o arquivo caso ele não exista. E a O_TRUNC apaga tudo o que havia antes. ]
```

**3. O número de reads e writes é igual? Por quê?**

```
[Sim, porque o programa lê um pedaço do arquivo de origem e escreve no arquivo de destino. Como essa operação se repete a cada pedaço, a quantidade de leituras e de escritas é exatamente a mesma.]
```

**4. Como você saberia se o disco ficou cheio?**

```
[Se o disco encher, a função de escrita write vai falhar. Ela não vai conseguir escrever tudo o que foi pedido.]
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
[Quando você não fecha um arquivo, o sistema continua achando que ele está em uso. Com o tempo, isso pode esgotar os recursos disponíveis.]
```

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
[As chamadas de sistema são a forma como um programa pede ajuda ao sistema do computador. Quando um programa quer fazer algo importante, como ler ou escrever um arquivo, ele precisa de permissão. Essa "mudança" de modos é o que permite ao sistema fazer o trabalho e depois devolver o controle ao programa com o resultado da operação.]
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
[File descriptors são números de identificação que o sistema usa para controlar os arquivos que um programa abre. Para cada novo arquivo que um programa abre, o sistema dá o próximo número disponível, que serve como uma referência para aquele arquivo.] 
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
[A relação entre o tamanho do buffer e a performance é quanto maior o buffer o programa é capaz de pegar mais dados de uma só vez. Isso significa que ele precisa fazer menos chamadas de sistema, o que economiza tempo sendo mais eficiente.]
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** __copia___

**Por que você acha que foi mais rápido?**

```
[Eu acho que por ser um programa de copia simples e feito para uma única função de copiar os dados. Ele acaba por ser mais rapido.]
```

---

## 📤 Entrega
Certifique-se de ter:
- [ ] Todos os códigos com TODOs completados
- [ ] Traces salvos em `traces/`
- [ ] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
