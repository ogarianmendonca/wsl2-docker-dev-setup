## Guia de Configuração de Ambiente de Desenvolvimento no WSL2 com Docker

### 1. Habilitar o Subsistema Windows para Linux (WSL) e a Plataforma de Máquina Virtual

1. Acesse: **Painel de Controle → Programas → Programas e Recursos → Ativar ou desativar recursos do Windows**.
2. Marque as opções:

   * **Subsistema Windows para Linux**
   * **Plataforma de Máquina Virtual**
3. Clique em **OK** e reinicie o computador se solicitado.

---

### 2. Alterar para o WSL 2

Abra o **PowerShell** e execute:

```powershell
wsl --set-default-version 2
```

> ⚠️ Caso não tenha o WSL 2 instalado, baixe e instale o pacote mais recente do WSL 2 disponibilizado pela Microsoft.

---

### 3. Instalar o Ubuntu

1. Abra a **Microsoft Store** e pesquise por **Ubuntu**.
2. Escolha a versão desejada (recomenda-se a versão LTS mais recente, ex.: Ubuntu 24.04).
3. Clique em **Adquirir** para iniciar a instalação.

---

### 4. Abrir o Ubuntu instalado

1. Abra o Ubuntu a partir do **menu Iniciar do Windows**.
2. Na primeira execução, siga as instruções para criar usuário e senha.

---

### 5. Instalar o Docker no Ubuntu

1. Atualize os pacotes:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
```

2. Configure o repositório do Docker:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

3. Instale o Docker e plugins:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

### 6. Verificar a instalação do Docker

Execute:

```bash
sudo docker run hello-world
```

Se a instalação estiver correta, uma mensagem de boas-vindas do Docker será exibida.

Teste:

```bash
sudo docker ps
```

> ⚠️ Se ainda não funcionar, reinicie o terminal do Ubuntu ou o WSL.

---

### 7. Configurar o Docker para rodar sem sudo

1. Adicione o usuário ao grupo `docker`:

```bash
sudo usermod -aG docker $USER
su - $USER
```

2. Teste sem `sudo`:

```bash
docker ps
```

> Se funcionar, a configuração está correta.

---

### 8. (OPCIONAL) Configurar o Docker Desktop (OPCIONAL)

1. Instale o [Docker Desktop para Windows](https://www.docker.com/products/docker-desktop/).
2. Nas configurações:

   * **Settings → General** → habilite **Use the WSL 2 based engine**
   * **Settings → Resources → WSL Integration** → habilite o Ubuntu instalado

---

### 9. Configuração da rede do Docker

1. Adicione o IP da bridge `docker0` ao `daemon.json`:

```bash
echo '{"bip": "192.168.1.5/24"}' | sudo tee -a /etc/docker/daemon.json
```

2. Abra o arquivo para conferir ou editar:

```bash
sudo nano /etc/docker/daemon.json
```

3. Reinicie o Docker:

```bash
sudo systemctl restart docker
```

4. Verifique a configuração:

```bash
ip addr show docker0
```

* Deve mostrar: `inet 192.168.1.5/24`

---

### 10. Instalação de PHP, extensões e Node.js

1. Adicione o repositório do PHP:

```bash
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
```

2. Instale PHP 8.3 e extensões necessárias, além de Git, Docker e Composer:

```bash
sudo apt-get install php8.3-gd php8.3 php8.3-dev php8.3-bcmath php8.3-sqlite php8.3-mysql php8.3-xml php8.3-soap php8.3-intl php8.3-curl php-pear composer libmcrypt-dev php8.3-mbstring php-xdebug php-ldap php8.3-sqlite
```
> **Observação:** Git, Docker e Docker Compose já foram instalados nos passos anteriores.

3. Escolha a versão do PHP (8.3) para compatibilidade com a versão do composer instalado.

```bash
sudo update-alternatives --config php
```

4. Instalar a extensão **mcrypt** compilando a partir do repositório oficial (Copie e execute todo o bloco abaixo junto):

```bash
git clone https://github.com/php/pecl-encryption-mcrypt.git
cd pecl-encryption-mcrypt
phpize
./configure
make
sudo make install
```

```bash
PARA PHP 8.3
cd pecl-encryption-mcrypt
phpize8.3
./configure --with-php-config=/usr/bin/php-config8.3
make
sudo make install
```

5. Habilitar a extensão **mcrypt** no PHP CLI:

```bash
echo "extension=mcrypt.so" | sudo tee -a /etc/php/8.4/cli/php.ini
```

```bash
PARA PHP 8.3
echo "extension=mcrypt.so" | sudo tee /etc/php/8.3/cli/conf.d/20-mcrypt.ini
```

> Se estiver usando Apache ou PHP-FPM, adicione a mesma linha em `/etc/php/8.3/apache2/php.ini` ou `/etc/php/8.3/fpm/php.ini`.

6. Instalar o **NVM** (Node Version Manager):

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

> **Observação:** Execute o comando indicado no final do script e **reinicie o terminal** (exemplo abaixo).
```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```
7. Instalar a versão 16.14.2 do Node.js via NVM:

```bash
nvm install v16.14.2
```

---

### 11. Usando o terminal no Ubuntu 24.04 (WSL2)


1. Na raiz do WSL, crie uma pasta para o projeto com um nome de sua preferência (exemplo: `PROJETOS`).

2. Dentro dessa pasta, **clone o repositório principal**:
   ```bash
   git clone <url-do-repositorio>
   ```

3. ⚠️ **Atenção**

* Todo o processo deve ser realizado **dentro do WSL2 (Ubuntu 24.04)**.  
* Isso inclui:  
   - **Clonar os projetos** (backend e frontend) diretamente no WSL2.  
   - **Executar com Docker** para subir os serviços necessários (backend, frontend e demais dependências).
