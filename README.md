# **2° Desafio Power BI - Bootcamp Dio/Santander 2023**

Transformação dos dados de uma base de dados fictícia, originadada de uma instância **MySQL Azure**, através do **Power Query** no Power BI. 

# Conjunto de Dados

O conjunto de dados utilizados denomina-se **azure_company** e contém informações fictícias de uma empresa distribuídas nas seguintes tabelas 

1. **Employee:** Contém informações dos empregados
2. **Dependent:** Informações dos dependentes dos empregados 
3. **Department:** Informações dos departamentos da empresa
4. **dept_locations:** Informações sobre a localização dos departamentos da empresa
5. **project:** Projetos em desenvolvimento
6. **works_on:** Informações sobre as designações dos funcionários


#  Compreensão do Desafio

Analisando as instruções estabelecidas no arquivo instruções.docx, percebo que o mesmo possui etapas que são comuns a todas as tabelas, enquanto outras restrigem-se a um grupo específico de informações. 

Desse modo, a metodologia utilizada será a de dividir o problema em tais grupos. Assim sendo, temos **correções globais** e **correções específicas.**

## Correções Globais

Nessa etapa serão realizadas as seguintes correções/modificações:

*(os passos equivalentes no instruções.docx estão em parênteses)*

1. Verificação de cabeçalhos e tipos de dados (1)
2. Modifique os valores monetários para o tipo double preciso (2)
3. Verifique a existência dos nulos e analise a remoção (3)
4. Separar colunas complexas (8)
5. Eliminar colunas desnecessárias (16)

## Correções específicas 
### Tabela *employee*

6. Os employees com nulos em Super_ssn podem ser os gerentes. Verifique se há algum colaborador sem gerente (4)
7. Mesclar consultas employee e departament para criar uma tabela employee com o nome dos departamentos associados aos colaboradores. A mescla terá como base a tabela employee. Após a mescla, eliminar as colunas desnecessárias. (9,10)
8.  Realize a junção dos colaboradores e respectivos nomes dos gerentes (11)
9.  Mescle as colunas de Nome e Sobrenome para ter apenas uma coluna definindo os nomes dos colaboradores (12)
10.  Agrupe os dados a fim de saber quantos colaboradores existem por gerente (15)

### Tabela *department*

11. Verifique se há algum departamento sem gerente (5). Se houver, presumir existência dos dados e preenchê-los (6)
12. Mesclar os nomes de departamento e localização, justificando a necessidade de mesclar e a impossibilidade de atribuir. (13, 14)

### Tabela *works_on* 

13. Verificar o número de horas dos projetos (7)

# Transformação dos Dados

## 1. Verificação de Cabeçalhos e tipos de dados 

**Alterações na tabela employee**

* Colunas renomeadas
  * fName > Nome
  * mInit > Inicial sobrenome
  * lName > Último nome
  * Ssn > ID
  * Birthday > Nascimento
  * Address > Endereço
  * Sex > Sexo
  * Salary > Salário 
  * Super_ssn > ID Gerente
  * dno > No. Departamento
  * azure_company.department > Referencia a tabela department. Renomeada para departamento e exibida como a informação expandida `Dname`.
  * azure_company.dependent > Referencia a tabela dependent. Renomeada para Qtd Dependentes e exibida como a informação agregada `# contagem de Sssn` 
  * azure_company.works_on > Referencia a tabela de projetos. Renomeada para Participação em Projetos e exibida como a informação agregada `# contagem de azure_company.project`

* Alteração no tipo de dados:
  * Salário > Número decimal fixo
  * ID > Inteiro 
  * ID Gerente > Inteiro

* Colunas removidas:
  * azure_company.employee(Ssn)
  * azure_company.employee(Super_ssn)

**Alterações na tabela dependent**

* Colunas renomeadas: 
  * Essn > Id Funcionário
  * Name_dependent > Nome Dependente
  * Sex > Sexo 
  * Bdate > Nascimento
  * Relationship > Parentesco
* Alteração do tipo de dados
  * Id Funcionário > Inteiro
* Colunas removidas: 
  * azure_company.employee

**Alterações na tabela Department**

* Colunas renomeadas:
  * Dname > Departamento 
  * Dnumber > ID Departamento 
  * mgr_ssn > ID Gerente
  * Mgr_start_date > Data início Gerência
  * Dept_create_date > Criação
  * azure_company_dept_locations > Renomeado para localização. Exibição da informação expandida `dLocation`
  * azure_company.employee > Renomeada para Qtd Funcionários. Exibição da informação expandida `employee(Ssn)` com a informação agregada `contagem de employee(Ssn)`
  * azure_company.project > Renomeada para Qtd Projetos. Exibição da informação agregada `Contagem de pName`

**Alteração na tabela dept_locations**

* Colunas renomeadas:
  * Dnumber > ID Departamento
  * Dlocation > Localização

**Alterações na tabela project**

* Colunas renomeadas:
  * Pname > Projeto
  * Pnumber > ID Projeto
  * Plocation > Local
  * Dnum > ID Departamento
  * azure_company_department > Renomeada para Dpto Responsável. Exibição da informação expandida `dName`
* Coluna removida
  * azure_company_works_on 

**Alterações na tabela works_on**

* Colunas renomeadas:
  * Essn > ID Funcionário
  * Pno > ID Projeto
  * Hours > Horas
  * azure_company.project > Localização Projeto. Exibição da informação expandida `pLocation`
* Colunas removidas:
  * azure_company.employee
 
## 2. Modificação de Valores monetários para o tipo double preciso 

Realizado na primeira etapa, na planilha employees.

## 3. Análise de Nulos e Possível Remoção 

A verificação aponta o registro de nulos apenas na tabela *employees*, nas colunas ID Gerente e Departamento.

Isso nos leva à descoberta de algumas situações interessantes, isto é, a relação entre funcionários e gerentes presentes na tabela.

Todavia, ainda é possível facilitar a visualização dos dados com mais algumas transformações. Nesse caso, irei mesclar as colunas `Nome`, `Inicial Sobrenome` e `Último nome`, mantendo a primeira como atributo de coluna.

Agora, atualizo a coluna `ID Gerente` com o nome completo do funcionário, utilizando a função de substituir valores do Power Query. 

Desse modo, podemos renomear a nova coluna para `Gerente` e excluir a coluna `ID`

Com essas informações, verificamos que o Funcionário `James E Borg` gerencia dois outros funcionários que também são gerentes, mas não possui gerente. Considerando, ainda, sua remuneração conforme a coluna `Salary` e sua lotação no departamento `Headquarters`, posso concluir que o valor `null` exibido pode ser mantido, uma vez que a informação não se aplica para o cargo ocupado. 

Agora, filtrando as informações de `Departamento`, podemos ver que os funcionários gerenciados por `Franklin T Wong` e `Jennifer S Wallace` não possuem informação de departamento preenchida. Tomado como base a informação de seus gerentes, realizamos o preenchimento desta informação com a ferramenta de substituição, eliminando os valores nulos da tabela.

Porém, a substituição direta dos valores `null` em `Departamento` prejudicaria a qualidade da informação, pois todos os dados seriam modificados. Assim sendo, melhor excluir a coluna `Departamento`, utilizando seu atributo na coluna `No. Departamento` e substituindo os valores correspondentes, sendo 1 para Headquarters, 4 para Administration e 5 para Research.

## 4. Separar colunas complexas

Visando a atomicidade das informações, passamos a separar colunas que possuam muitas informações em suas células. 

Dentre as colunas que restam na base de dados, a problemática é `Endereço` da tabela `Employees`. Para isso, realizamos uma série de transformações, separando Logradouro, Cidade e Estado, removendo os delimitadores da esquerda e direita e integrando as colunas conforme necessário. 

## 5. Eliminar colunas desnecessárias

Remoção das colunas de ID restantes

## 6. Os employees com nulos em Super_ssn podem ser os gerentes. Verifique se há algum colaborador sem gerente

Etapa realizada em conjunto ao passo 3. 

## 7. Mesclar consultas employee e departament para criar uma tabela employee com o nome dos departamentos associados aos colaboradores. A mescla terá como base a tabela employee. Após a mescla, eliminar as colunas desnecessárias. 

Efetuada mescla das tabelas `employee` e `department` numa nova tabela denominada `employee_department`. 

Removidas as colunas sobressalentes `Nascimento, Logradouro, Cidade, Estado, Sexo, Salary, Qtd Depedentes, Participação em Projetos` e `azure_company.department`

## 8.  Realize a junção dos colaboradores e respectivos nomes dos gerentes 

Realizado no passo 7.

## 9.  Mescle as colunas de Nome e Sobrenome para ter apenas uma coluna definindo os nomes dos colaboradores 

Realizado no passo 3.

## 10.  Agrupe os dados a fim de saber quantos colaboradores existem por gerente

Na nova tabela `employee_department` é criada uma tabela dinâmica partindo da coluna `Gerente`, exibindo então uma contagem de quantos funcionários são gerenciados e em quais departamentos.


## 11. Verifique se há algum departamento sem gerente. Se houver, presumir sua existência e preenchê-los 

Realizado no passo 3. 

## 12. Mesclar os nomes de departamento e localização, justificando a necessidade de mesclar e a impossibilidade de atribuir.

Na tabela `departmament` a mescla das colunas `Localização` e `Departamento` serve ao propósito de *enxugar* a informação da tabela e diferenciar os registros. 

Todavia, não seria possível fazer isso através de mera atribuição, uma vez que existem mais de um departamento `Research` e o Power Query **não permite** a alteração individual de células.


## 13. Verificar o número de horas dos projetos

Atualização do número de horas para inteiros, refletindo a duração total de um projeto. Correção de IDs, remoção e correção do valor 0 para refletir a duração do projeto associado.


# Considerações Finais

Embora a base de dados utilizada para este exemplo seja fictícia, as questões propostas permitem explorar diversas ferramentas do Power Query para Extração, Transformação e Carregamento de Dados, num verdadeiro **pipeline ETL.** 

As informações agora corrigidas e tratas podem ser usadas com confiança pelo analista para elaborar relatórios e *dashboards* que sejam pertinentes ao negócio. 

# A fazer
 - [ ] Elaborar painéis e relatórios com base nas informações tratadas
 - [ ] Publicar resultados no Power BI Service

