# 🔐 LicenseGuard - Sistema de Gestão de Licenças

Sistema completo de gestão e validação de licenças para produtos digitais (sites, bots, aplicações). Controle, valide e monitore licenças em tempo real com arquitetura moderna React + FastAPI + MongoDB.

---

## 📋 Índice

1. [Visão Geral](#visão-geral)
2. [Planos e Funcionalidades](#planos-e-funcionalidades)
3. [Arquitetura](#arquitetura)
4. [Como Funciona a Validação](#como-funciona-a-validação)
5. [Integrando em seus Produtos](#integrando-em-seus-produtos)
6. [Exemplos de Código](#exemplos-de-código)
7. [API Reference](#api-reference)
8. [Credenciais de Teste](#credenciais-de-teste)

---

## 🎯 Visão Geral

O LicenseGuard é uma plataforma SaaS que permite:

- **Criar e gerenciar projetos** licenciáveis
- **Gerar licenças** vinculadas a domínios ou identificadores
- **Validar licenças remotamente** via API REST
- **Controlar acesso** com múltiplos níveis de permissão
- **Monitorar status** e expiração de licenças

---

## 💎 Planos e Funcionalidades

### 📦 Grátis (R$ 0/mês)
- ✅ Acesso a projetos gratuitos públicos
- ✅ Geração de licenças básicas
- ✅ Painel de controle simplificado
- ❌ Projetos pagos

### 🌟 VIP (R$ 99/mês)
- ✅ Acesso a projetos gratuitos
- ✅ **Acesso a projetos pagos públicos**
- ✅ Licenças ilimitadas
- ✅ Suporte prioritário

### 🏢 Empresarial (R$ 299/mês)
- ✅ **Crie seus próprios projetos privados**
- ✅ **Projetos exclusivos invisíveis para outros**
- ✅ Licenças ilimitadas
- ✅ Sistema completo de validação
- ✅ Gerenciamento total de projetos

### 👨‍💼 Admin
- ✅ Controle total do sistema
- ✅ Gerenciamento de usuários
- ✅ Criação de projetos públicos globais
- ✅ Acesso a todas as licenças

---

## 🏗️ Arquitetura

```
┌─────────────────┐
│   Frontend      │
│   (React)       │
│   Port 3000     │
└────────┬────────┘
         │
         │ HTTPS
         │
┌────────▼────────┐
│   Backend API   │
│   (FastAPI)     │
│   Port 8001     │
└────────┬────────┘
         │
         │
┌────────▼────────┐
│    MongoDB      │
│   Port 27017    │
└─────────────────┘
```

**Coleções MongoDB:**
- `users` - Usuários do sistema
- `projects` - Projetos licenciáveis
- `licenses` - Licenças geradas

---

## 🔍 Como Funciona a Validação

### Fluxo Completo

```
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│   Produto    │         │  LicenseAPI  │         │   MongoDB    │
│ (Site/Bot)   │         │   Backend    │         │              │
└──────┬───────┘         └──────┬───────┘         └──────┬───────┘
       │                        │                        │
       │  1. Lê prod.key        │                        │
       ├────────────────────────┼────────────────────────┤
       │                        │                        │
       │  2. POST /api/public/validate                   │
       │  {key, project_code,   │                        │
       │   domain_or_identifier}│                        │
       ├───────────────────────>│                        │
       │                        │                        │
       │                        │  3. Busca licença      │
       │                        ├───────────────────────>│
       │                        │                        │
       │                        │  4. Valida:            │
       │                        │  - Status ativo?       │
       │                        │  - Não expirou?        │
       │                        │  - Projeto correto?    │
       │                        │  - Domínio correto?    │
       │                        │<───────────────────────┤
       │                        │                        │
       │  5. Resposta (200/401) │                        │
       │<───────────────────────┤                        │
       │                        │                        │
       │  6. Permite/Bloqueia   │                        │
       │     acesso             │                        │
       └────────────────────────┴────────────────────────┘
```

---

## 🚀 Integrando em seus Produtos

### Passo 1: Criar Projeto no LicenseGuard

1. Acesse o painel (Admin ou Empresarial)
2. Vá em **Projetos** → **Novo Projeto**
3. Preencha:
   - **Nome**: "Meu Bot Premium"
   - **Código**: "P-BOT-001" (único)
   - **Tipo**: pago ou gratuito

### Passo 2: Gerar Licença

1. Vá em **Licenças** → **Gerar Nova Licença**
2. Selecione o projeto
3. Informe o **domínio** (ex: `meusite.com`) ou **identificador** (ex: `chat_id_123456`)
4. Defina validade (dias)
5. Clique em **Gerar Licença**
6. **Baixe o arquivo `prod.key`**

### Passo 3: Arquivo prod.key

O arquivo `prod.key` é um JSON com este formato:

```json
{
  "key": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "project_code": "P-BOT-001"
}
```

**⚠️ Importante:**
- Este arquivo deve estar na **raiz do produto** (site/bot)
- O cliente final recebe este arquivo após comprar
- Nunca compartilhe publicamente

### Passo 4: Implementar Validação

Veja exemplos de código abaixo para diferentes linguagens.

---

## 💻 Exemplos de Código

### 🐘 PHP (Websites)

```php
<?php
// validar_licenca.php

// 1. Carregar o arquivo prod.key
$prodKeyPath = __DIR__ . '/prod.key';

if (!file_exists($prodKeyPath)) {
    die("Erro: Arquivo prod.key não encontrado!");
}

$prodKey = json_decode(file_get_contents($prodKeyPath), true);
$licenseKey = $prodKey['key'];
$projectCode = $prodKey['project_code'];

// 2. Obter domínio atual
$currentDomain = $_SERVER['HTTP_HOST'];

// 3. Fazer requisição para API de validação
$apiUrl = "https://seu-licenceguard.com/api/public/validate";

$postData = json_encode([
    'key' => $licenseKey,
    'project_code' => $projectCode,
    'domain_or_identifier' => $currentDomain
]);

$ch = curl_init($apiUrl);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $postData);
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Content-Type: application/json',
    'Content-Length: ' . strlen($postData)
]);

$response = curl_exec($ch);
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
curl_close($ch);

// 4. Verificar resposta
if ($httpCode === 200) {
    $result = json_decode($response, true);
    
    if ($result['status'] === 'VALIDO') {
        // ✅ Licença válida - permitir acesso
        echo "Licença válida! Projeto: " . $result['project'];
        // Continue com a lógica do seu site
    } else {
        // ❌ Licença inválida
        die("Licença inválida!");
    }
} else {
    // ❌ Erro na validação
    $error = json_decode($response, true);
    die("Erro de licença: " . $error['detail']['motivo']);
}
?>
```

### 🐍 Python (Bots Telegram/Discord)

```python
# bot_validator.py
import json
import os
import requests
import sys

def validar_licenca():
    """
    Valida a licença do bot antes de iniciar.
    Retorna True se válida, False caso contrário.
    """
    
    # 1. Carregar prod.key
    prod_key_path = os.path.join(os.path.dirname(__file__), 'prod.key')
    
    if not os.path.exists(prod_key_path):
        print("❌ Arquivo prod.key não encontrado!")
        return False
    
    with open(prod_key_path, 'r') as f:
        prod_key = json.load(f)
    
    license_key = prod_key['key']
    project_code = prod_key['project_code']
    
    # 2. Obter identificador do bot (chat_id, user_id, etc)
    # Para Telegram, pode usar o bot_token como identificador
    bot_identifier = os.getenv('BOT_TOKEN', 'bot_instance_1')
    
    # 3. Fazer requisição de validação
    api_url = "https://seu-licenceguard.com/api/public/validate"
    
    payload = {
        'key': license_key,
        'project_code': project_code,
        'domain_or_identifier': bot_identifier
    }
    
    try:
        response = requests.post(api_url, json=payload, timeout=10)
        
        if response.status_code == 200:
            result = response.json()
            
            if result['status'] == 'VALIDO':
                print(f"✅ Licença válida! Projeto: {result['project']}")
                print(f"📅 Expira em: {result['expires_at']}")
                return True
            else:
                print("❌ Licença inválida!")
                return False
        else:
            error = response.json()
            print(f"❌ Erro de validação: {error['detail']['motivo']}")
            return False
            
    except Exception as e:
        print(f"❌ Erro ao validar licença: {str(e)}")
        return False

# Exemplo de uso no bot
if __name__ == "__main__":
    if not validar_licenca():
        print("Bot bloqueado devido a licença inválida!")
        sys.exit(1)
    
    # Iniciar bot aqui
    print("Iniciando bot...")
    # bot.run()
```

### 🟨 Node.js (Aplicações JavaScript)

```javascript
// licenseValidator.js
const fs = require('fs');
const path = require('path');
const axios = require('axios');

async function validarLicenca() {
    try {
        // 1. Carregar prod.key
        const prodKeyPath = path.join(__dirname, 'prod.key');
        
        if (!fs.existsSync(prodKeyPath)) {
            throw new Error('Arquivo prod.key não encontrado!');
        }
        
        const prodKey = JSON.parse(fs.readFileSync(prodKeyPath, 'utf8'));
        const { key: licenseKey, project_code: projectCode } = prodKey;
        
        // 2. Obter identificador (domínio, app_id, etc)
        const identifier = process.env.APP_DOMAIN || 'localhost';
        
        // 3. Validar na API
        const apiUrl = 'https://seu-licenceguard.com/api/public/validate';
        
        const response = await axios.post(apiUrl, {
            key: licenseKey,
            project_code: projectCode,
            domain_or_identifier: identifier
        });
        
        if (response.status === 200 && response.data.status === 'VALIDO') {
            console.log('✅ Licença válida!');
            console.log(`📦 Projeto: ${response.data.project}`);
            console.log(`📅 Expira em: ${response.data.expires_at}`);
            return true;
        }
        
        return false;
        
    } catch (error) {
        if (error.response) {
            const motivo = error.response.data?.detail?.motivo || 'Desconhecido';
            console.error(`❌ Licença inválida: ${motivo}`);
        } else {
            console.error(`❌ Erro ao validar: ${error.message}`);
        }
        return false;
    }
}

// Exemplo de uso
(async () => {
    const licencaValida = await validarLicenca();
    
    if (!licencaValida) {
        console.error('Aplicação bloqueada devido a licença inválida!');
        process.exit(1);
    }
    
    // Iniciar aplicação
    console.log('Iniciando aplicação...');
    // startApp();
})();

module.exports = { validarLicenca };
```

### ☕ Java (Aplicações Desktop)

```java
// LicenseValidator.java
import com.google.gson.Gson;
import com.google.gson.JsonObject;
import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.file.Files;
import java.nio.file.Paths;

public class LicenseValidator {
    
    private static final String API_URL = "https://seu-licenceguard.com/api/public/validate";
    
    public static boolean validarLicenca() {
        try {
            // 1. Carregar prod.key
            String prodKeyJson = new String(Files.readAllBytes(Paths.get("prod.key")));
            Gson gson = new Gson();
            JsonObject prodKey = gson.fromJson(prodKeyJson, JsonObject.class);
            
            String licenseKey = prodKey.get("key").getAsString();
            String projectCode = prodKey.get("project_code").getAsString();
            
            // 2. Obter identificador (pode ser MAC address, hostname, etc)
            String identifier = System.getenv().getOrDefault("APP_ID", "app_instance_1");
            
            // 3. Criar payload
            JsonObject payload = new JsonObject();
            payload.addProperty("key", licenseKey);
            payload.addProperty("project_code", projectCode);
            payload.addProperty("domain_or_identifier", identifier);
            
            // 4. Fazer requisição HTTP
            HttpClient client = HttpClient.newHttpClient();
            HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(API_URL))
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(gson.toJson(payload)))
                .build();
            
            HttpResponse<String> response = client.send(request, 
                HttpResponse.BodyHandlers.ofString());
            
            // 5. Verificar resposta
            if (response.statusCode() == 200) {
                JsonObject result = gson.fromJson(response.body(), JsonObject.class);
                
                if ("VALIDO".equals(result.get("status").getAsString())) {
                    System.out.println("✅ Licença válida!");
                    System.out.println("📦 Projeto: " + result.get("project").getAsString());
                    return true;
                }
            } else {
                JsonObject error = gson.fromJson(response.body(), JsonObject.class);
                System.err.println("❌ Licença inválida: " + 
                    error.getAsJsonObject("detail").get("motivo").getAsString());
            }
            
            return false;
            
        } catch (IOException | InterruptedException e) {
            System.err.println("❌ Erro ao validar licença: " + e.getMessage());
            return false;
        }
    }
    
    public static void main(String[] args) {
        if (!validarLicenca()) {
            System.err.println("Aplicação bloqueada devido a licença inválida!");
            System.exit(1);
        }
        
        System.out.println("Iniciando aplicação...");
        // Iniciar aplicação
    }
}
```

---

## 📡 API Reference

### Endpoint de Validação Pública

**POST** `/api/public/validate`

**Não requer autenticação** - Este endpoint é público para validação de produtos.

#### Request Body

```json
{
  "key": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "project_code": "P-BOT-001",
  "domain_or_identifier": "meusite.com"
}
```

#### Response - Sucesso (200)

```json
{
  "status": "VALIDO",
  "message": "Licença OK",
  "project": "Meu Bot Premium",
  "expires_at": "2025-12-31T23:59:59+00:00"
}
```

#### Response - Erro (401)

```json
{
  "detail": {
    "status": "INVALIDO",
    "motivo": "Domain Mismatch"
  }
}
```

**Possíveis motivos de falha:**
- `License key not found` - Chave não existe
- `License is inactive` - Licença desativada
- `License expired` - Licença expirada
- `Mismatched Project` - Código de projeto incorreto
- `Domain Mismatch` - Domínio/identificador não corresponde

---

## 🔐 Endpoints Autenticados

Todos os endpoints abaixo requerem **Bearer Token JWT** no header:

```
Authorization: Bearer <token>
```

### Autenticação

**POST** `/api/auth/login`

```json
{
  "email": "usuario@email.com",
  "password": "senha123"
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "user": {
    "id": "...",
    "email": "usuario@email.com",
    "access_level": "vip"
  }
}
```

### Projetos

- **GET** `/api/projects` - Listar projetos
- **POST** `/api/projects` - Criar projeto (Admin/Empresarial)
- **PUT** `/api/projects/{id}` - Atualizar projeto
- **DELETE** `/api/projects/{id}` - Deletar projeto

### Licenças

- **GET** `/api/licenses` - Listar licenças
- **POST** `/api/licenses/generate` - Gerar nova licença
- **PATCH** `/api/licenses/{id}/toggle` - Ativar/desativar
- **DELETE** `/api/licenses/{id}` - Deletar licença

### Usuários (Admin apenas)

- **GET** `/api/users` - Listar usuários
- **POST** `/api/users` - Criar usuário
- **DELETE** `/api/users/{id}` - Deletar usuário

---

## 🔑 Credenciais de Teste

### Acesso Admin
```
Email: admin@license.com
Senha: admin123
```

### Acesso Empresarial
```
Email: empresa@teste.com
Senha: empresa123
```

---

## 🛠️ Instalação e Desenvolvimento

### Requisitos

- Python 3.11+
- Node.js 18+
- MongoDB 5.0+
- Yarn

### Backend

```bash
cd backend
pip install -r requirements.txt

# Configurar .env
echo "MONGO_URL=mongodb://localhost:27017" > .env
echo "DB_NAME=license_db" >> .env
echo "SECRET_KEY=$(openssl rand -hex 32)" >> .env

# Iniciar
uvicorn server:app --reload --port 8001
```

### Frontend

```bash
cd frontend
yarn install

# Configurar .env
echo "REACT_APP_BACKEND_URL=http://localhost:8001" > .env

# Iniciar
yarn start
```

---

## 📊 Estrutura MongoDB

### Collection: users

```json
{
  "id": "unique_id",
  "email": "usuario@email.com",
  "password_hash": "hashed_password",
  "access_level": "admin|vip|gratis|empresarial"
}
```

### Collection: projects

```json
{
  "id": "unique_id",
  "name": "Meu Projeto",
  "project_code": "P-CODE-001",
  "type": "pago|gratuito",
  "ownerId": "user_id|null"
}
```

**Nota:** `ownerId` é `null` para projetos públicos do admin e contém o `user_id` para projetos empresariais privados.

### Collection: licenses

```json
{
  "id": "unique_id",
  "license_key": "uuid-v4",
  "userId": "owner_user_id",
  "projectId": "project_id",
  "domain_or_identifier": "meusite.com",
  "expiration_date": "2025-12-31T23:59:59",
  "is_active": true,
  "created_at": "2024-01-01T00:00:00"
}
```

---

## 🔒 Segurança

### Boas Práticas

1. **Nunca exponha** o arquivo `prod.key` publicamente
2. **Use HTTPS** sempre para validações
3. **Implemente cache** local para reduzir requisições
4. **Valide periodicamente** (ex: a cada 24h, não a cada request)
5. **Ofusque o código** de validação em produtos compilados
6. **Criptografe** o arquivo prod.key se possível

### Exemplo de Cache (PHP)

```php
<?php
// Validar apenas 1x por dia
$cacheFile = __DIR__ . '/license_cache.txt';
$cacheValidity = 86400; // 24 horas

if (file_exists($cacheFile)) {
    $cacheTime = filemtime($cacheFile);
    if ((time() - $cacheTime) < $cacheValidity) {
        // Cache válido
        $isValid = file_get_contents($cacheFile) === '1';
        if ($isValid) {
            return true;
        }
    }
}

// Validar na API
$isValid = validarLicencaNaAPI();

// Salvar cache
file_put_contents($cacheFile, $isValid ? '1' : '0');

return $isValid;
?>
```

---

## 🚨 Troubleshooting

### Erro: "License key not found"
- Verifique se a chave no `prod.key` está correta
- Confirme que a licença existe no painel

### Erro: "Domain Mismatch"
- O domínio/identificador deve ser **exatamente** o mesmo cadastrado
- Para testes locais, use `localhost` ou `127.0.0.1`

### Erro: "Mismatched Project"
- O `project_code` deve corresponder ao projeto da licença
- Verifique se não copiou o código errado

### Erro: "License expired"
- A licença passou da data de expiração
- Gere uma nova licença no painel

---

## 📞 Suporte

Para dúvidas ou problemas:

1. Verifique a documentação acima
2. Revise os exemplos de código
3. Teste com as credenciais fornecidas
4. Entre em contato com suporte

---

## 📄 Licença

Este sistema é proprietário. Uso comercial requer licença válida.

---

**Desenvolvido com ❤️ usando React, FastAPI e MongoDB**
