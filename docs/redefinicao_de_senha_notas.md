# Fluxo de Recuperação de Senha e Primeiro Acesso: SyncDesk

## 1. Disparo de E-mail (Responsabilidade: Backend)
O backend identifica o tipo de usuário e gera o e-mail com a URI e o token de recuperação específicos.

* **Se o usuário for Cliente:**
    * O backend envia um *Deep Link* configurado para acionar o aplicativo móvel.
    * Formato: `syncdesk://reset-password?token={token_gerado}`
* **Se o usuário for Atendente:**
    * O backend envia uma URL padrão para acesso via navegador web.
    * Formato: `https://app.syncdesk.pro/reset-password?token={token_gerado}`

## 2. Roteamento e Interceptação (Responsabilidade: Aplicações / SO)
O dispositivo do usuário processa o clique no link recebido.

* **Cliente (Mobile):** O sistema operacional (Android/iOS) intercepta o esquema de URI `syncdesk://`, abre o aplicativo em segundo plano e direciona para a tela correspondente à rota.
    * *Independência de Nomenclatura:* O esquema `syncdesk://` é estritamente uma configuração de roteamento interno (definido no `AndroidManifest.xml` via `<intent-filter>` no Android). O nome comercial ou de exibição do aplicativo para o usuário final nas lojas e no dispositivo é totalmente independente dessa configuração e não precisa ser "syncdesk".
* **Atendente (Web):** O navegador processa o protocolo `http://` e carrega a aplicação web no domínio especificado.

*Nota de Implementação Obrigatória:* Ambos os frontends (Web e Mobile) exigem a configuração explícita de uma rota `/reset-password`. Esta rota deve ser programada para ler a *query string* da URL e armazenar o valor do parâmetro `token` em memória assim que a tela for carregada.

## 3. Interface de Redefinição (Responsabilidade: Frontends)
O usuário visualiza a tela renderizada pela rota `/reset-password` e interage com o formulário.

1.  O aplicativo/web exibe os campos para o usuário digitar a nova senha e confirmá-la.
2.  O usuário preenche os dados e submete a operação.

## 4. Efetivação da Troca (Comunicação Front -> Back)
O frontend empacota os dados e os envia de volta para a validação final.

1.  O frontend recupera o `token` que foi extraído da URL no Passo 2.
2.  O frontend executa uma requisição HTTP para o endpoint de recuperação do backend.
3.  O corpo da requisição (*payload*) envia as duas informações necessárias para fechar o ciclo.

**Exemplo de Payload:**
```json
{
  "token": "abc123_extraido_da_url",
  "new_password": "senha_digitada_na_interface"
}
```

O backend processa a requisição, valida a integridade e validade do token, atualiza o hash da senha no banco de dados e retorna o status HTTP adequado para o frontend.
