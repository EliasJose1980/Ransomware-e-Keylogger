# Simulação de Malwares com Python — Fins Educacionais
 
> **Projeto de conclusão** da trilha de Segurança da Informação — Digital Innovation One (DIO)  
> **Este repositório possui fins estritamente educacionais. Nenhum código aqui deve ser utilizado em sistemas reais sem autorização explícita.**
 
---
 
## Índice
 
1. [Sobre o Projeto](#sobre-o-projeto)
2. [Ransomware Simulado](#-ransomware-simulado)
3. [Keylogger Simulado](#-keylogger-simulado)
4. [Estratégias de Defesa](#-estratégias-de-defesa)
5. [Reflexão Final](#-reflexão-final)
6. [Como Executar](#-como-executar)
7. [Referências](#-referências)
---
 
## Sobre o Projeto
 
Este projeto implementa e documenta o comportamento de dois tipos clássicos de malware — **Ransomware** e **Keylogger** — usando Python em ambiente 100% controlado e isolado. O objetivo não é criar ferramentas maliciosas, mas sim **compreender como esses ataques funcionam por dentro**, para que possamos construir defesas mais eficazes.
 
### Objetivos de Aprendizagem
 
| Objetivo | Status |
|----------|--------|
| Entender o funcionamento prático de Ransomware | ok |
| Entender o funcionamento prático de Keylogger | ok |
| Implementar criptografia com Fernet (AES) | ok |
| Implementar captura de eventos de teclado | ok |
| Simular exfiltração de dados por e-mail | ok |
| Documentar estratégias de defesa | ok |
 
### Estrutura do Repositório
 
```
malware-simulado-python/
├── README.md                  ← Este arquivo
├── ransomware_simulado.py     ← Script de ransomware educacional
├── keylogger_simulado.py      ← Script de keylogger educacional
├── requirements.txt           ← Dependências Python
└── images/
    ├── demo_ransomware.png       ← Screenshot da criptografia
    └── demo_keylogger.png        ← Screenshot da captura
```
 
---
 
## Ransomware Simulado
 
### O que é um Ransomware?
 
Um **ransomware** é um tipo de malware que **criptografa os arquivos da vítima** e exige pagamento (geralmente em criptomoeda) pela chave de descriptografia. É uma das ameaças mais lucrativas e devastadoras na história da cibersegurança.
 
```
Anatomia de um ataque de ransomware:
 
  Vítima abre          Ransomware         Chave enviada
  arquivo malicioso  → executa e         → ao servidor
  (phishing, USB,      varre o disco       do atacante
   exploit)
                           ↓
                    Arquivos são          Mensagem de
                    criptografados   →    resgate exibida
                    (.locked, .enc)
```
 
### Como o Script Funciona
 
O arquivo [`ransomware_simulado.py`](ransomware_simulado.py) simula as seguintes etapas:
 
#### 1 Criação de Arquivos de Teste
 
Cria uma pasta `arquivos_teste/` com arquivos `.txt` simulando dados corporativos sensíveis (relatórios, cadastros, etc.).
 
#### 2 Geração de Chave Criptográfica
 
```python
from cryptography.fernet import Fernet
 
chave = Fernet.generate_key()  # AES-128 em modo CBC + HMAC-SHA256
```
 
O algoritmo **Fernet** é uma implementação segura do AES com autenticação de mensagem. Em ataques reais, esta chave é enviada ao servidor do criminoso — **sem ela, os dados são irrecuperáveis**.
 
#### 3 Criptografia dos Arquivos
 
```python
fernet = Fernet(chave)
dados_criptografados = fernet.encrypt(dados_originais)
# Salva como arquivo.txt.locked e remove o original
```
 
Cada arquivo é lido, criptografado e renomeado com a extensão `.locked`. O arquivo original é removido.
 
#### 4 Mensagem de Resgate
 
O script cria um arquivo `README_RECUPERE_SEUS_ARQUIVOS.txt` na pasta criptografada — exatamente o que ransomwares reais fazem (ex.: WannaCry, REvil, LockBit).
 
#### 5 Descriptografia (Simulando o Pagamento)
 
```python
dados_originais = fernet.decrypt(dados_enc)
# Restaura o arquivo original
```
 
No laboratório, a chave é salva localmente para demonstrar a recuperação. Em ataques reais, sem pagar o resgate (e mesmo assim, sem garantia), os dados são perdidos para sempre.
 
### Casos Reais Estudados
 
| Ransomware | Ano | Impacto |
|---|---|---|
| **WannaCry** | 2017 | 200.000+ computadores em 150 países; NHS britânico paralisado |
| **NotPetya** | 2017 | US$ 10 bi em danos; considerado o ataque mais destrutivo da história |
| **Colonial Pipeline** | 2021 | Pipeline de combustível dos EUA parado; US$ 4,4 mi de resgate pago |
| **JBS Foods** | 2021 | Maior processadora de carnes do mundo; US$ 11 mi pagos |
 
---
 
## Keylogger Simulado
 
### O que é um Keylogger?
 
Um **keylogger** é um software (ou hardware) que **registra secretamente todas as teclas digitadas** pelo usuário. Pode capturar senhas, dados bancários, mensagens privadas e qualquer outro texto digitado.
 
```
Fluxo de um keylogger:
 
  Usuário digita     Hook de teclado    Buffer em
  qualquer coisa  →  captura o evento → memória
                                            ↓
                    Exfiltração       ←  Grava em
                    (e-mail, FTP,        arquivo .txt
                     HTTP POST)
```
 
### Como o Script Funciona
 
O arquivo [`keylogger_simulado.py`](keylogger_simulado.py) demonstra:
 
#### Captura via `pynput`
 
```python
from pynput import keyboard
 
def ao_pressionar_tecla(tecla):
    texto = formatar_tecla(tecla)
    buffer.append(texto)
    
with keyboard.Listener(on_press=ao_pressionar_tecla) as listener:
    listener.join()
```
 
A biblioteca `pynput` usa hooks do sistema operacional para interceptar eventos de teclado **antes** que cheguem ao aplicativo ativo.
 
#### Tratamento de Teclas Especiais
 
```python
mapa_especiais = {
    "Key.space":     " ",
    "Key.enter":     "\n[ENTER]\n",
    "Key.backspace": "[⌫]",
    # ...
}
```
 
Teclas especiais são mapeadas para representações legíveis, preservando o contexto do que foi digitado.
 
#### Registro com Timestamps
 
A cada 10 teclas, um timestamp é inserido no log — permitindo reconstruir **quando** cada coisa foi digitada.
 
#### Thread de Envio Periódico
 
```python
thread_envio = threading.Thread(
    target=_thread_envio_periodico,  # Executa a cada 60s
    daemon=True
)
```
 
Uma thread separada simula o envio periódico do log por e-mail usando `smtplib` com SMTP autenticado (TLS).
 
#### Técnicas de Furtividade (Demonstração)
 
Keyloggers reais utilizam técnicas como:
- Rodar como serviço/daemon em background
- Nome de processo disfarçado (ex.: `svchost.exe`, `chrome_helper`)
- Sem janela visível (`pythonw.exe` no Windows)
- Persistência via `HKCU\...\Run` (registro do Windows) ou crontab
> **No laboratório**, o script roda em modo visível e possui tecla de saída (F12) para fins de transparência.
 
---
 
## Estratégias de Defesa
 
Esta seção documenta as principais contramedidas para cada tipo de ameaça.
 
### Contra Ransomware
 
#### Backup 3-2-1
A regra mais importante em segurança de dados:
- **3** cópias dos dados
- **2** mídias diferentes (HD externo + nuvem)
- **1** cópia offsite (fora do local físico principal)
> Um backup que nunca foi testado **não é um backup** — pratique a restauração!
 
#### Princípio do Menor Privilégio
Usuários e processos devem ter **apenas as permissões necessárias** para suas funções. Um ransomware que infecta uma conta sem privilégios de administrador tem alcance muito mais limitado.
 
#### Segmentação de Rede
Dividir a rede em segmentos isolados (VLANs, microsegmentação) impede que o ransomware se propague lateralmente pela organização — limitando o "raio de explosão" de um ataque.
 
#### Patch Management
Vulnerabilidades não corrigidas são a principal porta de entrada. O WannaCry explorou o **EternalBlue** (MS17-010) — uma vulnerabilidade do Windows que tinha patch disponível havia meses.
 
#### EDR e Análise Comportamental
Soluções de **Endpoint Detection and Response** detectam comportamentos suspeitos (ex.: processo criptografando centenas de arquivos em segundos) e bloqueiam antes da conclusão do ataque.
 
#### Sandboxing de E-mails
Anexos suspeitos são abertos em ambientes isolados antes de chegar à caixa de entrada do usuário. O conteúdo malicioso é detectado sem risco real.
 
---
 
### Contra Keyloggers
 
#### Autenticação Multifator (MFA)
Mesmo que a senha seja capturada, **o atacante ainda precisa do segundo fator** (token, biometria, SMS). É a defesa mais eficaz contra keyloggers.
 
```
Senha capturada pelo keylogger: ✓
Código TOTP do celular:         ✗ (atacante não tem acesso)
Resultado: login bloqueado!
```
 
#### Gerenciadores de Senha com Autopreenchimento
Ferramentas como **Bitwarden**, **1Password** e **KeePass** preenchem credenciais automaticamente sem que o usuário "precise digitar" a senha — tornando keyloggers ineficazes para capturar senhas.
 
#### Antivírus com Proteção em Tempo Real
Keyloggers registrados como software são detectados por assinaturas e heurística. Manter o antivírus atualizado é fundamental.
 
#### Teclado Virtual para Dados Sensíveis
Bancos brasileiros como Itaú, Bradesco e BB exigem o teclado virtual justamente para contornar keyloggers de software — as teclas do mouse não são registradas da mesma forma.
 
#### Monitoramento de Processos
Ferramentas como **Process Monitor** (Windows) ou `auditd` (Linux) permitem identificar processos com comportamento suspeito de hook de teclado.
 
---
 
### Defesas Universais
 
| Camada | Controle | Tecnologia |
|--------|----------|------------|
| **Humana** | Conscientização | Treinamentos, phishing simulado |
| **Endpoint** | Antivírus / EDR | CrowdStrike, Defender, Kaspersky |
| **Rede** | Firewall / IDS/IPS | pfSense, Snort, Suricata |
| **Aplicação** | Sandboxing | Cuckoo Sandbox, Any.run |
| **Dados** | Backup 3-2-1 | Veeam, Backblaze, AWS S3 |
| **Identidade** | MFA | Authenticator, YubiKey |
| **Processo** | Patch Management | WSUS, Ansible, Automox |
 
---
 
## Reflexão Final
 
### O que aprendi neste projeto
 
**Sobre Ransomware:**
 
A parte mais impactante foi perceber que a criptografia — uma tecnologia criada para **proteger** dados — é usada como arma. O Fernet (AES-128) que usei no laboratório é praticamente indistinguível do que ransomwares reais empregam. Isso reforça que a criptografia por si só é neutra; o que importa é **quem controla as chaves**.
 
O maior aprendizado foi entender por que a regra 3-2-1 de backups é tão crítica: se você tem um backup offline e recente, um ransomware se torna apenas um incidente de TI, não uma catástrofe.
 
**Sobre Keylogger:**
 
Foi revelador descobrir que a captura de teclas é uma funcionalidade **legítima** do sistema operacional (usada por softwares de acessibilidade, por exemplo). Keyloggers maliciosos exploram essa mesma interface — o que torna a detecção por assinatura menos eficaz e eleva a importância do MFA.
 
Entender que a **senha é apenas um fator** e que MFA pode anular completamente um keylogger me fez valorizar muito mais essa tecnologia.
 
**Sobre Defesa:**
 
Ambos os malwares dependem de algum vetor inicial (phishing, exploit, engenharia social). A conclusão mais importante é que **a segurança começa com as pessoas**: treinamento de conscientização e cultura de segurança são tão importantes quanto qualquer ferramenta técnica.
 
### A importância da ética em segurança
 
Todo conhecimento adquirido aqui tem um único propósito legítimo: **construir sistemas mais seguros e proteger pessoas e organizações**. O uso de qualquer técnica de ataque sem autorização expressa é ilegal e antiético.
 
Profissionais de cibersegurança que atuam com ética — pentesters, analistas de SOC, pesquisadores — são fundamentais para o ecossistema digital. Este projeto é o primeiro passo na compreensão de como pensar como um atacante para defender melhor.
 
---
 
## Como Executar
 
### Pré-requisitos
 
```bash
python --version   # Python 3.8+
```
 
### Instalação das Dependências
 
```bash
pip install -r requirements.txt
```
 
**`requirements.txt`:**
```
cryptography>=41.0.0
pynput>=1.7.6
```
 
### Executando o Ransomware Simulado
 
```bash
python ransomware_simulado.py
```
 
Escolha a opção **4** para a demonstração completa (cria → criptografa → descriptografa).
 
### Executando o Keylogger Simulado
 
```bash
python keylogger_simulado.py
```
 
Escolha a opção **1** para iniciar a captura. Pressione **F12** para encerrar.
 
> Execute apenas em sua própria máquina, em ambiente controlado.
 
---
 
## Referências
 
- [Python Docs](https://docs.python.org/3/)
- [Cryptography — Fernet](https://cryptography.io/en/latest/fernet/)
- [pynput — Keyboard Listener](https://pypi.org/project/pynput/)
- [smtplib — SMTP com Python](https://docs.python.org/3/library/smtplib.html)
- [MITRE ATT&CK — Ransomware T1486](https://attack.mitre.org/techniques/T1486/)
- [MITRE ATT&CK — Keylogging T1056.001](https://attack.mitre.org/techniques/T1056/001/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [CIS Controls v8](https://www.cisecurity.org/controls)
---
 

**Desenvolvido por Elias**  
Desafio de Projeto — Segurança da Informação  
Digital Innovation One (DIO) · 2025
 
*"O hacker ético usa o conhecimento do atacante para servir ao defensor."*
 
</div>
