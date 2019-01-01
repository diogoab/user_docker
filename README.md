# Instalação do Docker apartir do script SH

### O propósito do script de instalação é para uma conveniênte instalação de maneira rápida das versões mais recentes do Docker-CE nas distribuições Linux suportadas. Não é recomendado depender desse script para implantação em sistemas de produção. Para instruções mais detalhadas sobre como instalar nas distribuições suportadas, consulte as instruções de instalação em https://docs.docker.com/install/.

1. baixar e executar o script:
```
$ curl -fsSL https://get.docker.com -o get-docker.sh; sudo sh get-docker.sh
```

2. Conceder as devidas permissões para o usuário:
```
$ sudo usermod -aG docker $whoami
```