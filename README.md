# 🗂️ Projeto: Active Directory Lab

## 📄 Sumário

- [1. Objetivo do Projeto](#-1-objetivo-do-projeto)
- [2. Ambiente e Topologia](#-2-ambiente-e-topologia)
- [3. Etapas de Implementação e Configuração](#-3-etapas-de-implementação-e-configuração)
- [4. Desafios e Aprendizados](#-4-desafios-e-aprendizados)
- [5. Próximos Passos](#-5-próximos-passos)
- [6. Contato](#-6-contato)

---

## 📌 1. Objetivo do Projeto

Este repositório contém a documentação do meu homelab de **Active Directory**, criado para fins de estudo e prática em **tarefas típicas de suporte help desk**, como gerenciamento de usuários, grupos e permissões por meio do **Active Directory Users and Computers (ADUC)**.  
O objetivo principal foi me preparar para oportunidades como **estagiário** ou **suporte técnico júnior (Help Desk)**, simulando situações cotidianas de administração de contas em um ambiente corporativo.


---

## 🖥️ 2. Ambiente e Topologia

- **Hypervisor:** VMWare Workstation Pro
- **Servidor (DC01):** Windows Server 2022 Standard
- **Cliente (PC_Suporte):** Windows 11 Enterprise
- **Domínio:** `adlab.local`
- **Topologia de Rede:**
    - **Rede:** Rede Interna isolada para comunicação entre as VMs.
    - **Endereçamento IP do Servidor (DC01):** IP Fixo `192.168.50.10`

---

## ⚙️ 3. Etapas de Implementação e Configuração

A seguir estão os resumos de cada etapa. Para um passo a passo detalhado com todas as evidências, etapas e prints, **clique no título da seção correspondente**.

### [3.1. Instalação e Promoção do Controlador de Domínio 📄➡️](docs/01-Instalacao-e-Promocao.md)

O documento detalha a criação da VM, a instalação do Windows Server 2022 e suas configurações iniciais (IP estático e hostname). Em seguida, a role do Active Directory (AD DS) foi instalada e o servidor foi promovido a Controlador de Domínio, criando a nova floresta adlab.local.

![Promoção a DC](https://raw.githubusercontent.com/yshizw/homelab-active-directory/main/img/promodc.jpg)

### [3.2. Gerenciamento de Usuários e Grupos 📄➡️](docs/02-Gerenciamento-de-Usuarios.md)

Esta etapa detalha a criação de Unidades Organizacionais (OUs) para simular os departamentos da empresa, seguida da criação de usuários de teste e grupos de segurança. O documento também demonstra tarefas diárias de Help Desk, como o reset de senhas e o desbloqueio de contas de usuários.

![Criação de Usuários no ADUC](https://raw.githubusercontent.com/yshizw/homelab-active-directory/main/img/ou_user_aduc.jpg)

### [3.3. Compartilhamento de Pastas e Permissões (File Server) 📄➡️](docs/03-Compartilhamento-de-Pastas.md)

Documenta a criação de um compartilhamento de rede central (a pasta Setores), com a permissão de compartilhamento configurada para Usuários Autenticados (Authenticated Users). Em seguida, demonstra a aplicação de permissões NTFS granulares: Leitura na pasta-pai (o "corredor") e Modificar na subpasta Financeiro, garantindo que cada departamento acesse apenas sua própria área.

![Configuração de Permissões NTFS](https://raw.githubusercontent.com/yshizw/homelab-active-directory/main/img/folder_permi.jpg)

### [3.4. Políticas de Grupo (GPO) 📄➡️](docs/04-Politicas-de-Grupo.md)

Detalha a implementação de múltiplas Políticas de Grupo (GPOs) para reforçar a segurança e automatizar o ambiente. Isso inclui a configuração das políticas de senha e bloqueio (Default Domain Policy), o mapeamento automático de unidades de rede para departamentos, a padronização de papéis de parede e a restrição de segurança avançada, como o bloqueio do PowerShell para usuários comuns.

![GPMC](https://raw.githubusercontent.com/yshizw/homelab-active-directory/main/img/4.1.jpg)

---

## 💡 4. Desafios e Aprendizados

- **Desafio Encontrado:** (O mais complexo) A máquina cliente não conseguia se comunicar com o Controlador de Domínio, retornando erros de `DNS request timed out`, mesmo com a rede e o ping funcionando.
    - **Diagnóstico:** As verificações básicas de serviço e firewall estavam corretas. O problema foi isolado na forma como o servidor DNS processava as consultas.
    - **Solução:** O diagnóstico (realizado com assistência de IA) revelou que o problema não estava na zona de pesquisa direta, mas sim na ausência de uma **Zona de Pesquisa Reversa (Reverse Lookup Zone)** e de um registro **PTR**. A criação desses registros no servidor resolveu o problema de timeout imediatamente.
    - **[➡️ Veja o relatório técnico completo deste incidente aqui.](docs/05-Troubleshooting-DNS.md)**

- **Desafio Encontrado:** Usuários não conseguiam criar/modificar arquivos em suas pastas de departamento (ex: `Financeiro`), mesmo com a permissão NTFS de "Modificar" corretamente aplicada.
    - **Diagnóstico:** A investigação revelou um conflito de permissões em camadas. A Permissão de Compartilhamento (`Share Permission`) na pasta-pai (`Setores`) estava configurada de forma restritiva (`Read`), o que se sobrepôs à permissão NTFS permissiva (`Modify`) da subpasta.
    - **Solução:** A arquitetura foi corrigida: 1) A Permissão de Compartilhamento da pasta-pai foi alterada para `Authenticated Users` com `Full Control`. 2) A Permissão NTFS da pasta-pai (o "corredor") foi definida para `Read & Execute` (para navegação), resolvendo o conflito e garantindo que a segurança granular do NTFS nas subpastas funcionasse como esperado.


- **Desafio Encontrado:** Bloquear o PowerShell para usuários comuns, mas permitir para a equipe de TI, sem criar GPOs conflitantes.
    - **Diagnóstico:** A melhor prática não é criar duas políticas concorrentes, mas sim uma política com uma exceção.
    - **Solução:** Foi utilizada uma única GPO de bloqueio (`Disable PowerS`). Um grupo de exceção (`Enable PowerS`) foi criado, e o **Filtro de Segurança** da GPO foi editado para **`Negar`** a permissão de **`Apply group policy`** para este grupo, liberando o acesso da equipe de TI de forma eficiente.


- **Principais Aprendizados:**
    - **A regra de ouro das permissões:** A diferença crítica e a regra de precedência entre Permissões de Compartilhamento (a "porta da rua") e Permissões NTFS (a "porta da sala"). A permissão **mais restritiva** das duas é a que sempre vence.
    - **O poder do Filtro de Segurança em GPOs:** Aprendi a usar a aba "Delegação" para aplicar filtros de segurança, como `Negar` a aplicação de uma GPO para um grupo de exceção (ex: liberar o PowerShell para a TI), o que é uma prática muito mais limpa.
    - **A importância das ferramentas de diagnóstico:** O uso de `gpresult /r` no cliente para confirmar quais GPOs estão sendo aplicadas e o uso do `Visualizador de Eventos` (Event Viewer) no servidor para identificar IDs de falha específicos (como o `4771` para falhas de autenticação Kerberos) são essenciais para o troubleshooting.
    - **Melhores Práticas de AD:** Entendi por que a política de senhas do domínio deve ser editada na `Default Domain Policy` em vez de se criar uma nova, e por que o grupo `Authenticated Users` é superior ao `Everyone`.
    - **Utilização de Ferramentas**: A importância de usar todas as ferramentas disponíveis para o diagnóstico, incluindo assistentes de IA, para navegar e resolver problemas complexos (como a falta de uma zona de DNS reverso) que estão além do escopo de conhecimento inicial.

---

## 🚀 5. Próximos Passos

- [ ] **Automação com PowerShell:** Criar scripts básicos em PowerShell para automatizar tarefas, como a criação de múltiplos usuários a partir de um arquivo CSV.
- [ ] **Servidor de Impressão:** Implementar a função de Print Server e distribuir impressoras via GPO.

---

## 📞 6. Contato

* **LinkedIn:** `https://www.linkedin.com/in/markus-yoshizawa`
* **E-mail:** `markus.yoshizawa@gmail.com`
