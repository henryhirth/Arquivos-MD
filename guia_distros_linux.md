# Guia Completo: 25 Distros Linux para Programação e Estudo em VMs
### Para desenvolvedores com Fedora Workstation como host | AMD Ryzen 7 5825U | 32 GB RAM

---

> **Como usar este guia:** Leia primeiro a Visão Geral de Categorias para entender o mapa do território. Depois consulte cada distro individualmente. Termine pelo Plano de Estudos Sugerido para saber por onde começar.

---

## Sumário

1. [Visão Geral: Categorias de Distros](#categorias)
2. [Seu Hardware e KVM: O que você precisa saber](#hardware)
3. [As 25 Distros — Análise Profunda](#distros)
4. [Fedora no Fedora: Quando faz sentido?](#fedora-no-fedora)
5. [Plano de Estudos Sugerido](#plano)
6. [Tabela de Referência Rápida](#tabela)

---

## 1. Visão Geral: Categorias de Distros {#categorias}

Antes de entrar em cada distro individualmente, é fundamental entender que as distribuições Linux não existem num vácuo — elas pertencem a famílias e filosofias distintas que refletem escolhas técnicas profundas. Para um desenvolvedor estudando em VMs, essas diferenças são o objeto de estudo em si.

### Categoria A: Mainstream de Desktop (Base Ubuntu/Debian)
Ubuntu LTS, Debian Stable, Linux Mint, Pop!_OS, elementaryOS

**Por que importa para estudo:** São a maioria do mercado de servidores Linux (especialmente Ubuntu/Debian) e o ambiente que você encontrará em tutoriais, documentações de projetos open source e VPS de empresas. Entender o gerenciador `apt`, a estrutura `/etc/apt/sources.list`, o modelo de repositórios e o ciclo de releases deb é conhecimento de altíssima empregabilidade.

**Diferença filosófica central:** Debian prioriza estabilidade e liberdade; Ubuntu prioriza usabilidade e ciclo previsível; as derivadas (Mint, Pop, elementary) priorizam experiência de usuário em cima do Ubuntu.

### Categoria B: Servidor Estável Empresarial (Base RHEL)
RHEL, CentOS Stream, AlmaLinux, Rocky Linux, Oracle Linux

**Por que importa para estudo:** O mundo corporativo — especialmente bancos, telecomunicações, indústria e governo — roda em RHEL ou seus clones. Entender `dnf`/`yum`, SELinux no modo enforcing, `systemd` no estilo RHEL, certificações Red Hat (RHCSA/RHCE) e o modelo de suporte de longo prazo são habilidades altamente valorizadas no mercado empresarial.

**Diferença filosófica central:** Estabilidade extrema e suporte pago de longo prazo. O pacote que está na versão X vai ficar na versão X por anos, com apenas patches de segurança. Isso é o oposto de rolling release.

### Categoria C: Rolling Release para Desenvolvedores Avançados
Arch Linux, EndeavourOS, Manjaro, openSUSE Tumbleweed, Gentoo

**Por que importa para estudo:** Rolling release significa que você sempre tem o software mais recente. Para desenvolvedores que trabalham com linguagens e ferramentas que evoluem rápido (Rust, Go, LLVM, kernels novos), isso é uma vantagem real. Mais importante: distros como Arch te ensinam Linux "de baixo pra cima" — você monta o sistema do zero e entende cada camada.

**Diferença filosófica central:** Arch = você decide tudo. openSUSE = rolling com ferramentas enterprise-grade (YaST). Gentoo = você compila tudo do fonte, controle total sobre flags de otimização.

### Categoria D: Servidor/Cloud Moderno
Fedora Server, Ubuntu Server, Debian Server, Alpine Linux, NixOS

**Por que importa para estudo:** Simular servidores reais é fundamental para qualquer desenvolvedor que vai trabalhar com backend, DevOps, cloud ou infraestrutura. Alpine é especial: é a base de imagens Docker, então entendê-la te ajuda a entender containers. NixOS é revolucionário na forma como gerencia configuração declarativa.

### Categoria E: Especializadas
Kali Linux (segurança), Parrot OS (segurança/privacidade), Ubuntu Studio (criação), Clear Linux (Intel/performance), Void Linux (independente/minimalista)

**Por que importa para estudo:** Cada uma dessas existe porque há um nicho de problemas que as distros generalistas resolvem mal. Kali e Parrot são obrigatórias para quem estuda segurança. Clear Linux ensina otimização de performance. Void Linux tem um sistema de init diferente (runit) e um gerenciador de pacotes único (xbps).

---

## 2. Seu Hardware e KVM: O que você precisa saber {#hardware}

### AMD Ryzen 7 5825U e Virtualização

O Ryzen 7 5825U possui **AMD-V (SVM)** habilitado, que é o suporte a virtualização por hardware da AMD. Isso significa que o KVM/libvirt vai funcionar nativamente no seu Fedora sem nenhuma configuração especial além de verificar que SVM está habilitado na BIOS (geralmente já está).

```bash
# Verifique se a virtualização AMD está ativa
grep -m1 svm /proc/cpuinfo
# ou
lscpu | grep Virtualization
# Deve mostrar: Virtualization: AMD-V
```

### GPU Radeon Integrada e VMs

A GPU integrada Radeon (parte do APU Ryzen 5825U — provavelmente Radeon RX Vega 8 ou similar) usa o driver `amdgpu` open source no kernel Linux. Para VMs de desenvolvimento e estudo, isso é excelente por três razões:

1. **Nenhuma VM de desenvolvimento precisa de aceleração GPU real.** Para programação backend, DevOps, ciência de dados em CPU, compilação de código — o display virtual (VirtIO ou QXL) é suficiente.
2. **GPU passthrough seria complexo com GPU integrada.** Mas você não precisa disso para o seu objetivo.
3. **O driver amdgpu é open source e estável.** O host Fedora vai gerenciar a GPU perfeitamente; as VMs simplesmente usam emulação de vídeo.

### 32 GB de RAM: Sua Maior Vantagem

Com 32 GB de RAM, você pode rodar múltiplas VMs simultaneamente com conforto:

| Perfil de VM | RAM sugerida | Quantas simultâneas |
|---|---|---|
| VM minimalista (Alpine, Debian minimal) | 512 MB – 1 GB | 10–15 |
| VM de servidor (Ubuntu Server, Rocky) | 1–2 GB | 8–12 |
| VM de desktop (Ubuntu Desktop, Fedora) | 2–4 GB | 4–6 |
| VM pesada (compilação, IA/dados) | 4–8 GB | 2–4 |

### Drivers AMD em VMs — O que realmente importa

Quando você roda uma distro em uma VM no KVM, o guest (a VM) não acessa diretamente sua GPU Radeon. Ele usa um adaptador de vídeo **emulado** (QXL para SPICE, ou VirtIO-GPU para melhor performance). Isso significa que:

- **Drivers AMD no guest são irrelevantes para display.** Não importa se a distro suporta bem hardware AMD ou não — ela nunca vai "ver" sua GPU real.
- **O que importa é o suporte a VirtIO.** VirtIO são drivers paravirtualizados que dão melhor performance de disco, rede e vídeo em KVM. Todas as distros modernas têm suporte a VirtIO no kernel.
- **Para desenvolvimento nativo (não VM):** Aí sim, o suporte AMD importa. Mas todas as distros modernas (kernel 5.15+) têm excelente suporte ao `amdgpu`.

### KVM vs GNOME Boxes vs virt-manager

| Ferramenta | Uso ideal | Limitações |
|---|---|---|
| **GNOME Boxes** | Criar/rodar VMs rapidamente, desktop simples | Poucas opções de configuração, sem controle fino de CPU/memória/rede |
| **virt-manager** | Controle completo: múltiplas CPUs, redes customizadas, snapshots, storage pools | Interface mais complexa, mas muito mais poderosa |
| **virsh** (linha de comando) | Automação, scripting, CI/CD | Curva de aprendizado mais alta |
| **Vagrant + libvirt** | Ambientes reproduzíveis via código (Infrastructure as Code) | Requer instalar plugin `vagrant-libvirt` |

**Recomendação:** Use GNOME Boxes para VMs simples e experimentos rápidos. Use virt-manager para VMs que você vai manter, configurar em detalhes e usar para projetos sérios. Aprenda virsh quando quiser automatizar.

---

## 3. As 25 Distros — Análise Profunda {#distros}

---

### 01. Ubuntu LTS (22.04 / 24.04)

**Base:** Debian | **Modelo:** Fixed Release (LTS = 5 anos suporte) | **Package Manager:** APT + dpkg

#### Resumo de Propósito
Ubuntu LTS é a distro mais usada no mundo para servidores, desenvolvimento e tutoriais. Quando um tutorial de programação diz "instale com `apt install`", ele foi escrito pensando em Ubuntu ou Debian. Quando uma documentação de projeto open source dá instruções de instalação em Linux, Ubuntu é o alvo implícito.

#### Para que tipo de programação é melhor
- **Web development (Node.js, Python, Ruby, PHP):** Ubuntu tem PPAs (Personal Package Archives) que permitem instalar versões específicas de runtimes sem quebrar o sistema. `apt-add-repository ppa:deadsnakes/python3` para múltiplas versões de Python, por exemplo.
- **DevOps e Cloud:** AWS, GCP e Azure usam Ubuntu como imagem padrão em muitos casos. Aprender Ubuntu é aprender o ambiente de produção real.
- **Ciência de dados e IA:** A maioria dos notebooks Jupyter, guias de instalação de TensorFlow/PyTorch e ambientes Conda são testados primariamente em Ubuntu.
- **Desenvolvimento de aplicativos com Snap:** Ubuntu é o inventor e maior usuário de Snaps. Para entender essa forma de empacotamento (e suas controvérsias), Ubuntu é o lugar certo.

#### Tipo de desenvolvedor que mais se beneficia
- **Iniciantes:** A quantidade de tutoriais, respostas no Stack Overflow e documentação assume Ubuntu. Quando algo dá errado, a probabilidade de você achar a solução exata para Ubuntu é muito maior.
- **Devs que trabalham com equipes:** Se a sua empresa usa Ubuntu em servidores, faz todo sentido desenvolver no mesmo ambiente.
- **Cientistas de dados:** Ambientes Conda, ambientes virtuais Python, JupyterHub — tudo isso é mais testado em Ubuntu.

#### Vantagens para uso em VM (KVM/libvirt)
- Instalador moderno com suporte nativo a VirtIO (o instalador detecta automaticamente que está em KVM).
- Imagens oficiais de cloud (`ubuntu-22.04-server-cloudimg-amd64.img`) são otimizadas para VM e iniciam em segundos com cloud-init.
- Suporte excelente a QEMU Guest Agent (`qemu-guest-agent`), que permite ao host comunicar com o guest, fazer snapshots consistentes e ver o IP.
- Documentação extensiva para uso em KVM/libvirt da própria Canonical.

#### Desvantagens e armadilhas para estudo em VM
- **Snap em VMs:** O daemon `snapd` consome memória e CPU mesmo quando você não usa snaps. Para VMs com pouca RAM (1–2 GB), isso é um desperdício. Você pode desabilitar o snapd ou usar a versão minimal do instalador.
- **GNOME desktop completo é pesado:** 4+ GB de RAM para ter uma experiência fluida. Para estudo de servidor, use a versão Server (sem GUI), que roda confortavelmente em 512 MB.
- **Versões antigas de pacotes no LTS:** Python 3.10 pode estar disponível enquanto o mercado usa 3.12. Isso te força a aprender a usar PPAs ou pyenv/nvm/asdf para versões específicas — o que, na verdade, é uma habilidade útil.

#### Mini-projetos de estudo recomendados
1. **Deploy de aplicação web completa:** Node.js + PostgreSQL + Nginx como reverse proxy. Aprenda a configurar systemd units para cada serviço.
2. **CI/CD local com Gitea + Drone:** Simule um pipeline de integração contínua inteiramente dentro da VM.
3. **Ubuntu Server + cloud-init:** Aprenda a provisionar VMs automaticamente via cloud-init configs (YAML). Isso é o que AWS/GCP fazem internamente.
4. **AppArmor profiles:** Ubuntu usa AppArmor (não SELinux). Entenda como criar perfis de segurança para aplicações.
5. **LXD containers dentro da VM:** Ubuntu tem integração nativa com LXD. Você pode rodar containers LXD dentro de uma VM KVM para estudar múltiplos níveis de isolamento.

---

### 02. Debian Stable

**Base:** Original | **Modelo:** Fixed Release (muito conservador, ~2 anos entre versões) | **Package Manager:** APT + dpkg

#### Resumo de Propósito
Debian é o pai de Ubuntu e de centenas de outras distros. É a definição de estabilidade e conservadorismo técnico. Um servidor Debian bem configurado pode rodar por anos sem precisar de intervenção. Não é a distro mais empolgante para desenvolvimento, mas é uma das mais respeitadas e ensinantes.

#### Para que tipo de programação é melhor
- **Administração de sistemas e DevOps:** Entender Debian te dá uma base sólida para entender Ubuntu, Linux Mint e qualquer distro baseada em Debian. O `apt` em Debian é o `apt` "canônico".
- **Servidores web de longa vida:** Ambientes PHP/MySQL/Apache ou LEMP stack que precisam de estabilidade máxima. Debian é amplamente usado em hosting compartilhado.
- **Programação em C/C++ para sistemas:** Os pacotes de desenvolvimento em Debian são muito bem organizados (`build-essential`, `libssl-dev`, etc.).
- **Software livre estrito:** Debian só inclui software verdadeiramente livre por padrão. Para estudar a diferença entre free/non-free software, Debian é a escola.

#### Tipo de desenvolvedor que mais se beneficia
- **Intermediários que querem entender a "base":** Usar Debian te força a entender mais sobre gerenciamento de pacotes, repositórios e configuração manual porque há menos "magia" automática que o Ubuntu adiciona.
- **Sysadmins e DevOps:** Debian é muito usado em servidores bare-metal e VPS de baixo custo.

#### Vantagens para uso em VM
- **Levíssimo:** Uma instalação Debian minimal sem GUI consome ~300–400 MB de RAM e ~2 GB de disco. Você pode rodar 6–8 VMs Debian simultâneas no seu hardware.
- **Imagens netinstall:** O instalador netinstall tem ~400 MB e baixa apenas o que precisa. Perfeito para VMs.
- **Previsibilidade:** Debian Stable não vai surpreender você com mudanças. O sistema que você configurou hoje vai se comportar da mesma forma em 6 meses.
- **Debian cloud images:** Assim como Ubuntu, tem imagens cloud prontas para KVM.

#### Desvantagens e armadilhas
- **Pacotes muito antigos:** Debian Stable trava versões quando é lançado. Debian 12 (Bookworm, lançado em 2023) tem Python 3.11, PostgreSQL 15, GCC 12 — não são as versões mais recentes, mas são sólidas.
- **Menos hand-holding:** Se algo não funciona, há menos tutoriais específicos para Debian do que para Ubuntu. Você precisa saber traduzir instruções entre distros.
- **Drivers proprietários não incluídos por padrão:** Firmware proprietário (como drivers Wi-Fi Broadcom) não vem habilitado por padrão. Mas em VM isso não importa — você usa drivers VirtIO de qualquer forma.

#### Mini-projetos de estudo recomendados
1. **Instalação manual e configuração de servidor LAMP/LEMP:** Sem painéis de controle, tudo na mão.
2. **Gerenciamento de versões com backports:** Aprenda a usar `debian-backports` para obter versões mais recentes de pacotes específicos sem quebrar o sistema.
3. **Configuração de segurança com fail2ban + UFW:** Monte uma fortaleza básica de servidor.
4. **Comparação de pacotes Debian vs Ubuntu:** Instale o mesmo software nas duas VMs e compare configurações padrão, localização de arquivos, diferenças de comportamento.
5. **Debian Testing ou Unstable em VM:** Aprenda a usar diferentes branches (stable/testing/unstable/experimental) e entenda os trade-offs de cada um.

---

### 03. Arch Linux

**Base:** Original | **Modelo:** Rolling Release | **Package Manager:** pacman + AUR (yay/paru)

#### Resumo de Propósito
Arch Linux é frequentemente mal compreendido. Não é sobre dificuldade ou "status" — é sobre uma filosofia de minimalismo radical e transparência. Você instala apenas o que precisa, e você sabe exatamente o que está instalado e por quê. O resultado é um sistema que você entende completamente porque você o construiu do zero.

O Arch Wiki é lendário: é a melhor documentação técnica de Linux que existe, e é útil mesmo quando você não usa Arch.

#### Para que tipo de programação é melhor
- **Desenvolvimento de software em geral com ferramentas de ponta:** Arch tem sempre a versão mais recente de compiladores, runtimes e ferramentas. GCC 14, LLVM 18, Python 3.13, Rust latest — tudo disponível imediatamente após o release upstream.
- **Contribuição para projetos open source:** Se você quer contribuir para projetos que usam as versões mais recentes das dependências, Arch é o melhor ambiente.
- **Programação de sistemas (C, C++, Rust, Assembly):** O controle fino sobre quais versões de toolchains você usa é valioso.
- **Tudo que o AUR oferece:** O AUR (Arch User Repository) tem praticamente qualquer pacote que você imaginar — ferramentas de niche, versões beta de software, utilitários obscuros. É imenso.

#### Tipo de desenvolvedor que mais se beneficia
- **Intermediários a avançados** que querem entender profundamente como o Linux funciona. A instalação manual de Arch (sem scripts automáticos) te ensina: particionamento, bootloader, locale, fuso horário, rede, gerenciamento de usuários — tudo do zero.
- **Devs curiosos** que gostam de ter controle total e não querem "surpresas" de um sistema que muda coisas automaticamente.
- **Mantenedores de pacotes** que querem aprender como o empacotamento Linux funciona (PKGBUILDs são mais simples de escrever que debs ou RPMs).

#### Vantagens para uso em VM
- **Extremamente leve quando configurado corretamente:** Uma VM Arch sem GUI pode rodar em 256–512 MB de RAM.
- **Controle total sobre o que está instalado:** Sem daemons indesejados rodando em background.
- **Rolling release em VM é perfeito:** Você pode manter a VM sempre atualizada sem reinstalar. `pacman -Syu` uma vez por semana e você tem o sistema mais recente.
- **Suporte a VirtIO no kernel padrão:** O kernel Arch (`linux` package) inclui todos os módulos VirtIO necessários.

#### Desvantagens e armadilhas
- **Instalação manual tem curva de aprendizado:** Mas isso é o ponto. Considere usar o `archinstall` script (disponível desde 2021) para uma primeira instalação guiada, e depois fazer uma manual para aprender.
- **Atualizações podem quebrar o sistema:** Rolling release significa que ocasionalmente uma atualização introduz um problema. Em VM, isso é ótimo para estudo — você aprende a diagnosticar e corrigir. Em produção, seria problemático.
- **Menos "estável" por definição:** Se você precisa de um ambiente de estudo que "sempre funciona igual", Arch não é a escolha. Mas se você quer aprender a lidar com a realidade de um sistema em constante evolução, é perfeito.
- **AUR requer atenção:** Pacotes AUR são mantidos pela comunidade e não são auditados pela equipe Arch. Sempre leia o PKGBUILD antes de instalar.

#### Mini-projetos de estudo recomendados
1. **Instalação manual completa do zero:** Siga o Arch Wiki sem scripts. Aprenda particionamento com `fdisk`/`parted`, instalação do bootloader GRUB, configuração de locale e timezone. Faça isso 3 vezes até fluir naturalmente.
2. **Criar um PKGBUILD:** Empacote um software simples (script Python, ferramenta de linha de comando) como pacote Arch. Aprenda a estrutura de um PKGBUILD.
3. **Configurar um ambiente de desenvolvimento mínimo:** Instale apenas o que você precisa para um projeto específico. Aprenda a diferença entre dependências de runtime e de build.
4. **Testar uma atualização que quebrou algo e corrigir:** Arch tem um histórico de mudanças (`/var/log/pacman.log`). Pratique downgrade de pacotes com `pacman -U`.
5. **Usar o AUR:** Instale `yay` ou `paru`, explore o AUR, aprenda a lidar com conflitos de dependências.

---

### 04. EndeavourOS

**Base:** Arch | **Modelo:** Rolling Release | **Package Manager:** pacman + AUR

#### Resumo de Propósito
EndeavourOS é frequentemente descrito como "Arch Linux com um instalador". Mas é mais que isso: é uma distro que te dá a experiência Arch com menos atrito inicial, sem tirar o que torna o Arch valioso — o controle, o minimalismo e o AUR.

Diferente de Manjaro (que tem repositórios próprios e atrasa as atualizações), EndeavourOS usa os repositórios oficiais do Arch diretamente. A única adição é um instalador gráfico (Calamares) e uma comunidade acolhedora.

#### Para que tipo de programação é melhor
Praticamente idêntico ao Arch. A diferença é que EndeavourOS é mais acessível para quem quer a experiência Arch sem passar pelo rito de iniciação da instalação manual.

#### Tipo de desenvolvedor que mais se beneficia
- **Quem quer Arch mas está aprendendo:** EndeavourOS é o melhor caminho intermediário entre "distro fácil" e "Arch puro".
- **Devs que já conhecem Linux** mas não querem gastar horas na instalação manual do Arch.

#### Vantagens para uso em VM
- Instalador Calamares funciona bem em VM (detecta ambiente virtual).
- Após instalação, é idêntico ao Arch — mesmos benefícios.
- Levíssimo com opção de instalar sem DE (Bare Desktop).

#### Desvantagens e armadilhas
- A "facilidade" de instalação pode te fazer perder o aprendizado que a instalação manual do Arch proveria.
- **Recomendação:** Use EndeavourOS para ter um Arch funcional rapidamente, mas depois faça uma instalação manual do Arch puro em uma segunda VM para o aprendizado.

#### Mini-projetos de estudo recomendados
- Os mesmos do Arch, mas use EndeavourOS como "Arch de backup" quando quiser algo que funcione rapidamente.
- **Comparação direta:** Tenha uma VM EndeavourOS e uma VM Arch puro, ambas atualizadas ao mesmo tempo. Compare o comportamento das atualizações.

---

### 05. Fedora Workstation (e variantes)

**Base:** Original (upstream do RHEL) | **Modelo:** Semi-rolling (releases a cada 6 meses) | **Package Manager:** DNF + RPM

#### Resumo de Propósito
Você já usa Fedora como host! Fedora é a distribuição "upstream" do Red Hat Enterprise Linux. O que entra no Fedora hoje, entra no RHEL daqui a 2–3 anos. Isso a torna extremamente relevante para entender a direção técnica do ecossistema enterprise.

Fedora é a distro onde as inovações do Linux aparecem primeiro: Wayland foi padrão no Fedora antes de qualquer outra distro, PipeWire também, Btrfs como filesystem padrão, systemd-resolved, etc.

#### Para que tipo de programação é melhor
- **Desenvolvimento enterprise Java/Jakarta EE:** O ecossistema Red Hat (WildFly, Quarkus, Keycloak) é desenvolvido e testado principalmente em Fedora/RHEL.
- **Containers e Kubernetes:** Podman (alternativa rootless ao Docker) foi criado pela Red Hat e é o padrão no Fedora. OpenShift (Kubernetes enterprise) é Red Hat. Estudar containers no Fedora é estudar o ambiente onde muitas empresas deployam.
- **Programação de sistemas em C/C++/Rust:** GCC e Clang no Fedora são sempre muito atualizados. O projeto Fedora é patrocinado pela Red Hat, que tem grande investimento em toolchains.
- **SELinux:** Fedora usa SELinux no modo enforcing por padrão. Entender SELinux é uma habilidade diferenciada no mercado enterprise.

#### Vantagens para uso em VM
- **VMs Fedora no host Fedora = máxima compatibilidade:** Mesmo kernel, mesmos drivers, mesmas tecnologias de virtualização.
- **Fedora Server edition:** Instalação mínima de servidor sem GUI, ideal para VMs de estudo de DevOps.
- **Fedora CoreOS:** Uma variante imutável projetada para containers/Kubernetes. Excelente para estudar arquiteturas modernas de deployment.
- **Fedora Spins:** Existem spins com diferentes DEs (KDE, XFCE, i3) e spins especializados (Fedora Security Lab, Fedora Scientific).

#### Desvantagens e armadilhas
- Se seu objetivo é aprender as diferenças entre distros, usar Fedora na VM te dá menos aprendizado "diferencial" do que usar Ubuntu ou Arch.
- Ciclo de release de 6 meses significa que você precisa atualizar a versão da VM periodicamente.
- Algumas ferramentas de terceiros têm melhor suporte para Ubuntu/Debian — instaladores que assumem `apt` podem dar trabalho em Fedora.

#### Mini-projetos de estudo recomendados
1. **SELinux mastery:** Aprenda a criar políticas SELinux personalizadas. Estude os contextos de segurança (`ls -Z`), use `audit2allow` para criar políticas a partir de logs de negação.
2. **Podman e containers rootless:** Fedora é o melhor ambiente para aprender Podman. Crie pods, use Podman Compose, compare com Docker.
3. **Quarkus microservices:** O framework Java nativo para cloud da Red Hat. Compile para binário nativo com GraalVM.
4. **Fedora CoreOS em VM:** Provisionamento via Ignition configs (o Butane/Ignition é a versão Fedora do cloud-init). Simule um nó de um cluster Kubernetes.
5. **RPM packaging:** Crie um pacote RPM do zero. Aprenda a diferença entre spec files e PKGBUILDs/debs.

---

### 06. CentOS Stream

**Base:** RHEL | **Modelo:** Rolling (midstream do RHEL) | **Package Manager:** DNF + RPM

#### Resumo de Propósito
CentOS Stream é hoje o que alimenta o desenvolvimento do RHEL. Ao contrário do antigo CentOS (que era um clone do RHEL já lançado), o CentOS Stream está *à frente* do RHEL — é onde o RHEL é desenvolvido continuamente.

**Contexto histórico importante para o estudo:** Em 2020, a Red Hat descontinuou o CentOS 8 como clone do RHEL e transformou em CentOS Stream. Isso causou polêmica e levou à criação de Rocky Linux e AlmaLinux como substitutos "clones do RHEL". Entender essa história te dá contexto político e técnico do ecossistema Linux enterprise.

#### Para que tipo de programação é melhor
- **DevOps e administração de sistemas enterprise:** Se você vai trabalhar com RHEL no ambiente corporativo, CentOS Stream é o seu ambiente de testes.
- **Desenvolvimento de pacotes RPM** que serão distribuídos para clientes RHEL.

#### Vantagens para uso em VM
- Pacotes mais recentes que Rocky/Alma (está à frente do RHEL em desenvolvimento).
- Gratuito, suportado pela Red Hat.
- Boa documentação.

#### Desvantagens e armadilhas
- Não é um clone estável do RHEL — é o ambiente de desenvolvimento do RHEL. Pode ter problemas que ainda não chegaram ao RHEL.
- Para simular ambientes de produção RHEL estáveis, Rocky Linux ou AlmaLinux são melhores.

---

### 07. Rocky Linux

**Base:** RHEL (clone 1:1) | **Modelo:** Fixed Release | **Package Manager:** DNF + RPM

#### Resumo de Propósito
Rocky Linux foi criado por Gregory Kurtzer (um dos fundadores do CentOS original) como resposta direta ao fim do CentOS 8. É um clone binário do RHEL — os pacotes são compilados a partir dos fontes do RHEL, com apenas a remoção de marcas registradas Red Hat.

**Para o estudo:** Rocky Linux é a melhor opção para simular exatamente um servidor RHEL sem pagar a licença Red Hat. O que você aprende em Rocky Linux se aplica diretamente ao RHEL.

#### Para que tipo de programação é melhor
- **Administração de sistemas enterprise:** SELinux enforcing, `firewalld`, `cockpit` (painel web de administração), estrutura de diretórios RHEL.
- **Certificações Red Hat (RHCSA/RHCE):** Os exames são em RHEL, mas Rocky Linux é o ambiente de estudo gratuito perfeito.
- **Aplicações que serão deployadas em RHEL:** Teste a compatibilidade do seu software com o ambiente RHEL sem precisar de licença.

#### Vantagens para uso em VM
- Imagens KVM/QCOW2 oficiais disponíveis para download.
- Leve para servidor (sem GUI): 1 GB de RAM é suficiente.
- Muito estável — não haverá surpresas de atualizações.
- QEMU Guest Agent incluído e funcional.

#### Desvantagens e armadilhas
- **Versões de pacotes muito antigas:** Python 3.9, Node.js 16, PHP 7.x — em RHEL 9 (base do Rocky 9). Para desenvolvimento com versões recentes, você precisa usar AppStream modules ou SCL (Software Collections).
- **AppStream modules são confusos para iniciantes:** Entender como instalar "Python 3.11" em Rocky Linux é uma curva de aprendizado por si só.
- Não é para quem quer experimentar tecnologias de ponta.

#### Mini-projetos de estudo recomendados
1. **Simulação de ambiente corporativo completo:** Rocky Linux como servidor de aplicação, PostgreSQL, Nginx, SELinux enforcing. Configure tudo sem desabilitar SELinux.
2. **Estudo para RHCSA:** Use o livro "RHCSA Red Hat Enterprise Linux 9" e pratique em VM Rocky Linux.
3. **Cockpit administration:** O painel web de administração Cockpit vem instalado e é muito poderoso. Aprenda a gerenciar serviços, logs, storage e rede via web UI.
4. **Firewalld zones:** A configuração de firewall no mundo RHEL é diferente do ufw/iptables direto. Aprenda a trabalhar com zonas no firewalld.

---

### 08. AlmaLinux

**Base:** RHEL (clone 1:1) | **Modelo:** Fixed Release | **Package Manager:** DNF + RPM

#### Resumo de Propósito
AlmaLinux é o outro grande clone do RHEL, criado pela CloudLinux (empresa de hosting). É funcionalmente equivalente ao Rocky Linux para a maioria dos usos.

**Diferença prática Rocky vs Alma:** Rocky Linux é frequentemente preferido pela comunidade acadêmica e por sysadmins que vieram do CentOS original. AlmaLinux tem backing corporativo mais forte (CloudLinux Inc.) e é muito usado em ambientes de hosting. Para estudo, são intercambiáveis.

**Escolha Rocky ou Alma, não ambos.** A menos que você queira estudar especificamente as diferenças de empacotamento entre as duas, elas são equivalentes para fins de aprendizado RHEL.

---

### 09. openSUSE Leap

**Base:** SUSE (derivado do RHEL indiretamente) | **Modelo:** Fixed Release | **Package Manager:** zypper + RPM

#### Resumo de Propósito
openSUSE Leap é a versão estável do openSUSE e tem uma relação com SUSE Linux Enterprise (SLE) semelhante à relação Fedora/RHEL — é o upstream da versão enterprise paga. Leap usa pacotes do SLE mais contribuições da comunidade.

O openSUSE é notável por ter a **melhor ferramenta de administração gráfica do mundo Linux:** o YaST (Yet Another Setup Tool). Para administração de sistemas, YaST é impressionante.

#### Para que tipo de programação é melhor
- **Administração de sistemas em ambientes SUSE enterprise:** SUSE tem forte presença em empresas alemãs/europeias e em ambientes SAP (SAP é certificado principalmente em SUSE/RHEL).
- **Programação em ambientes SAP HANA:** SAP HANA roda primariamente em SUSE Linux. Para desenvolvedores que trabalham com SAP, entender openSUSE/SLE é fundamental.
- **Configuração declarativa com YaST:** YaST tem módulos para configurar praticamente tudo via GUI ou TUI.

#### Vantagens para uso em VM
- YaST funciona perfeitamente em VM, inclusive via terminal (YaST TUI).
- OBS (Open Build Service): a infraestrutura de empacotamento do openSUSE é extremamente avançada e disponível publicamente.
- Bom suporte a KVM nas imagens oficiais.

#### Desvantagens e armadilhas
- **zypper é diferente de apt/dnf:** A sintaxe é diferente (`zypper install`, `zypper search`, `zypper update`). Para quem vem de apt/dnf, há uma pequena curva.
- **Menor comunidade** do que Ubuntu/Arch/Fedora. Menos recursos de ajuda online em português.
- **Menos relevante no mercado brasileiro:** SUSE tem presença menor no Brasil comparado à Red Hat e Canonical.

#### Mini-projetos de estudo recomendados
1. **YaST exploration completa:** Configure rede, firewall, usuários, partições, serviços — tudo via YaST. Compare com como você faria isso no Fedora.
2. **OBS (Open Build Service):** Use o OBS público (build.opensuse.org) para entender como é a infraestrutura de CI/CD de uma distro. Build um pacote simples.
3. **AppArmor no openSUSE:** openSUSE usa AppArmor (como Ubuntu), não SELinux. Compare os dois sistemas de MAC (Mandatory Access Control).

---

### 10. openSUSE Tumbleweed

**Base:** SUSE | **Modelo:** Rolling Release (com testes automatizados) | **Package Manager:** zypper + RPM

#### Resumo de Propósito
Tumbleweed é a versão rolling do openSUSE e é única no mercado: é um rolling release **com testes automatizados rigorosos antes de cada atualização**. O sistema openQA testa automaticamente centenas de casos de uso antes de qualquer atualização chegar ao usuário.

Isso a torna a rolling release mais estável que existe — mais estável que Arch, mais estável que Manjaro. Você tem software recente sem o risco de instabilidade que Arch às vezes apresenta.

#### Para que tipo de programação é melhor
- Tudo que openSUSE Leap oferece, mas com software atualizado.
- **Desenvolvimento com versões recentes** em um ambiente RPM (diferente de Arch que usa outro sistema de empacotamento).
- Excelente para quem quer uma rolling release mas com mais confiança na estabilidade.

#### Mini-projetos de estudo recomendados
1. **Comparar rolling releases:** Mantenha VMs Tumbleweed e Arch simultâneas. Acompanhe o timing das atualizações de um software específico (como GCC ou Python) nas duas distros. Veja como o openQA retarda atualizações para garantir estabilidade.
2. **Transactional Updates:** Tumbleweed suporta atualizações transacionais com btrfs + snapper. Aprenda a fazer rollback de uma atualização problemática.

---

### 11. Linux Mint

**Base:** Ubuntu LTS | **Modelo:** Fixed Release | **Package Manager:** APT + dpkg

#### Resumo de Propósito
Linux Mint é famosa como a "distro para iniciantes" mas isso a subestima. Mint é uma curadoria muito cuidadosa do Ubuntu: remove o Snap por padrão (usando Flatpak como alternativa), tem o desktop Cinnamon (desenvolvido pela própria equipe Mint) e foca em estabilidade de desktop.

**Para programação em VM:** Mint tem valor limitado como distro de estudo avançado porque é essencialmente Ubuntu com um desktop diferente. Para servidor, use Ubuntu Server ou Debian diretamente.

#### Quando é útil em VM
- Para estudar o desktop Cinnamon (único e bem desenvolvido).
- Para entender as escolhas de curadoria que o Linux Mint faz em relação ao Ubuntu.
- Para comparar gestão de software sem Snap vs com Snap.

#### Mini-projetos de estudo recomendados
1. **Comparação Snap vs Flatpak:** Instale o mesmo software (VS Code, por exemplo) via Snap no Ubuntu e via Flatpak no Mint. Compare tamanho, performance, integração com o sistema.
2. **Mint Update Manager:** Estude o sistema de levels de atualização do Mint (1-5) e como ele gerencia a estabilidade.

---

### 12. Pop!_OS

**Base:** Ubuntu LTS | **Modelo:** Fixed Release | **Package Manager:** APT + dpkg + Pop Shop

#### Resumo de Propósito
Pop!_OS foi criada pela System76 (fabricante de hardware Linux) e tem foco em desenvolvedores e científistas. Inclui ferramentas de desenvolvimento pré-instaladas, tem um gerenciador de janelas tiling (auto-tiling GNOME), e famosa pela excelente integração com hardware NVIDIA (mas você não precisa disso).

**Para estudo em VM:** Similar ao Linux Mint — é Ubuntu com opiniões diferentes. O valor de estudo é menor do que Ubuntu puro ou Debian. Mas se você usa Pop!_OS no mundo real e quer simular seu ambiente de desenvolvimento, faz sentido.

#### Mini-projetos de estudo recomendados
1. **Pop Shell tiling window manager:** Entenda como o tiling WM funciona. Compare com i3wm "puro" em uma VM Arch.
2. **Pop!_OS em VM como ambiente de desenvolvimento Python/Data Science:** Aproveite que vem com muita coisa pré-configurada para ciência de dados.

---

### 13. Manjaro Linux

**Base:** Arch | **Modelo:** Rolling Release atrasado (~2 semanas) | **Package Manager:** pacman + AUR + repos Manjaro

#### Resumo de Propósito
Manjaro busca ser "Arch para iniciantes" — tem instalador gráfico, drivers proprietários automáticos, e os pacotes do Arch são atrasados ~2 semanas para testes extras.

**Controvérsia importante:** A comunidade Arch tem opiniões negativas sobre Manjaro, principalmente porque Manjaro tem seus próprios repositórios que não são 100% compatíveis com o AUR. Isso pode causar conflitos difíceis de resolver.

**Para estudo:** Se você quer Arch acessível, EndeavourOS é a escolha melhor que Manjaro. Manjaro tem sua própria camada de complexidade (repos próprios, atualização de certificados famosa por atrasos) que pode confundir mais do que ensinar.

---

### 14. NixOS

**Base:** Original | **Modelo:** Estável (NixOS 24.x) ou Unstable (rolling) | **Package Manager:** Nix

#### Resumo de Propósito
NixOS é a distro mais filosoficamente diferente de todas as outras nesta lista. É construída em torno do **Nix package manager** e do conceito de **configuração declarativa e reproduzível**.

Em NixOS, você descreve o estado completo do sistema em um arquivo de configuração (`/etc/nixos/configuration.nix`). O Nix garante que esse estado seja reproduzível — se você pegar o mesmo arquivo de configuração e aplicar em outra máquina, você tem o mesmo sistema.

Isso resolve um problema fundamental de "funciona na minha máquina": com NixOS/Nix, a configuração do ambiente é code, versionável em Git, e 100% reproduzível.

#### Para que tipo de programação é melhor
- **DevOps e Infrastructure as Code avançado:** NixOS é a expressão máxima de "infrastructure as code" no nível de SO.
- **Desenvolvimento com múltiplas versões de linguagens/tools:** `nix-shell` te permite entrar em um ambiente isolado com versões específicas de qualquer ferramenta sem afetar o sistema.
- **Flakes (Nix Flakes):** O sistema moderno do Nix para gerenciar ambientes de desenvolvimento reproduzíveis.
- **Contribuição para projetos que usam Nix:** Muitos projetos open source fornecem `flake.nix` para ambiente de desenvolvimento.

#### Tipo de desenvolvedor que mais se beneficia
- **Devs avançados** que sofreram com o problema de "funciona na minha máquina" e querem uma solução radical.
- **Pesquisadores e cientistas** que precisam de reprodutibilidade absoluta em ambientes computacionais.
- **DevOps/SRE** que querem levar o conceito de IaC para o nível do sistema operacional.

#### Vantagens para uso em VM
- Ideal para VM porque você pode versionar a `configuration.nix` no Git e recriar o ambiente exato a qualquer momento.
- **Rollback instantâneo:** Cada geração do sistema NixOS fica no bootloader. Atualização quebrou algo? Boot na geração anterior.
- Levíssimo quando configurado minimamente.

#### Desvantagens e armadilhas
- **Curva de aprendizado alta:** A linguagem Nix (usada no `configuration.nix`) é uma linguagem funcional de configuração com quirks únicos. É muito diferente de qualquer outro arquivo de configuração Linux.
- **Documentação fragmentada:** A documentação oficial é incompleta em muitas áreas. O NixOS discourse e o GitHub são frequentemente mais úteis.
- **Flakes são "experimental" mas são o futuro:** Há uma divisão na comunidade entre o sistema "antigo" e Flakes, o que confunde iniciantes.
- **Não segue o FHS:** O NixOS não segue o Filesystem Hierarchy Standard. Isso quebra binários pré-compilados que assumem `/usr/lib` etc.

#### Mini-projetos de estudo recomendados
1. **Home Manager:** Configure todo o ambiente do usuário (dotfiles, packages, configurações de apps) com Nix Home Manager. Versione tudo em Git.
2. **Nix Flakes para ambiente de desenvolvimento:** Crie um `flake.nix` para um projeto Python ou Node.js que define exatamente as versões de todas as dependências. Mostre que funciona em outra máquina/VM sem nenhuma configuração adicional.
3. **NixOS em VM como servidor:** Configure um servidor web completo inteiramente via `configuration.nix`. Destrua e recrie a VM usando apenas o arquivo de configuração.
4. **Comparação com Docker:** Compare o problema que NixOS resolve com o que Docker resolve. Quais são as sobreposições? Quais são as diferenças fundamentais?

---

### 15. Alpine Linux

**Base:** Original | **Modelo:** Semi-rolling (versões estáveis + edge) | **Package Manager:** apk

#### Resumo de Propósito
Alpine Linux é a base padrão das imagens Docker oficiais de milhares de projetos. É minuscula (~5 MB como imagem base), segura por design (usa musl libc e busybox ao invés de glibc e coreutils GNU) e extremamente eficiente.

**Para desenvolvimento:** Se você trabalha com containers, entender Alpine é entender o ambiente onde seu código roda em produção. Muitos bugs de "funciona no dev, quebra em prod" surgem de diferenças entre o sistema do dev (glibc) e a imagem de produção Alpine (musl libc).

#### Para que tipo de programação é melhor
- **Desenvolvimento e otimização de imagens Docker:** Se sua imagem base é Alpine, você precisa entender Alpine.
- **Segurança:** Alpine é usada como base para ferramentas de segurança por ser minimalista e ter pouca superfície de ataque.
- **Programação em C para ambientes minimalistas:** A diferença entre musl libc e glibc é relevante para programadores C que precisam de binários portáveis.
- **Embedded/IoT (simulado):** Alpine simula um ambiente constrito como você encontraria em sistemas embarcados.

#### Vantagens para uso em VM
- **Extremamente leve:** Instalação mínima em ~50–100 MB de disco, VM roda com 128–256 MB de RAM.
- Você pode rodar dezenas de VMs Alpine simultâneas no seu hardware.
- Ótima para simular múltiplos servidores em uma topologia de rede.

#### Desvantagens e armadilhas
- **musl libc vs glibc:** Muitos softwares foram escritos e testados com glibc. Em Alpine com musl, você pode ter comportamentos diferentes ou falhas de compilação.
- **Busybox:** Alpine usa busybox para os comandos Unix básicos (ls, cat, grep, sed...). O busybox tem menos features que as versões GNU. Scripts bash que dependem de extensões GNU podem falhar.
- **Não use Alpine para desenvolvimento geral:** Use para estudar especificamente containers e ambientes minimalistas.

#### Mini-projetos de estudo recomendados
1. **Criar imagens Docker mínimas:** Use Alpine como base. Aprenda a usar multi-stage builds para criar imagens Go, Python, Node.js de tamanho mínimo.
2. **musl vs glibc:** Compile um programa C simples que usa funções de string ou rede. Compare o comportamento entre Alpine (musl) e Ubuntu (glibc).
3. **Network topology simulation:** Use múltiplas VMs Alpine para simular uma rede com router, servidor web, servidor de banco de dados, e cliente. Configure interfaces de rede manualmente.

---

### 16. Kali Linux

**Base:** Debian Testing | **Modelo:** Rolling | **Package Manager:** APT + dpkg

#### Resumo de Propósito
Kali Linux é a distro de segurança ofensiva mais famosa do mundo, mantida pela Offensive Security. Vem com centenas de ferramentas de pentest, análise forense, engenharia reversa e hacking ético pré-instaladas.

**Mal-entendido comum:** Kali não é para uso como distro de desenvolvimento do dia a dia. É uma toolbox especializada. A Offensive Security explicitamente recomenda usar Kali apenas quando você precisa das ferramentas de segurança.

#### Para que tipo de programação é melhor
- **Segurança da informação e pentest:** Se você quer aprender desenvolvimento seguro (secure coding), entender Kali te dá perspectiva de como o atacante pensa.
- **CTF (Capture The Flag):** Competições de segurança. Kali tem todas as ferramentas necessárias.
- **Web application security:** Burp Suite (versão community), SQLmap, Nikto — ferramentas de teste de aplicações web.
- **Análise de código malicioso:** Ferramentas forenses para análise de binários.

#### Vantagens para uso em VM
- **VM é o ambiente ideal para Kali:** Você quer isolamento quando fazendo pentest. Uma VM de Kali pode ser destruída e recriada facilmente.
- Imagens VMs oficiais da Offensive Security disponíveis em formato QCOW2 para KVM.
- Snapshots antes de cada teste te permitem voltar a um estado limpo.

#### Desvantagens e armadilhas
- **Não é para iniciantes como sistema principal:** Kali roda como root por padrão (nas versões antigas), tem configurações de segurança afrouxadas por design (para ferramentas de pentest funcionarem). Isso é o oposto do que você quer em um sistema de desenvolvimento.
- **Foco estreito:** Se você não está estudando segurança, o tempo com Kali é melhor gasto em outras distros.

#### Mini-projetos de estudo recomendados
1. **Ambiente de lab isolado:** Crie uma rede virtual isolada (libvirt internal network) com uma VM Kali como "atacante" e uma VM Ubuntu/Metasploitable como "alvo". Nunca teste em redes reais.
2. **OWASP WebGoat:** Instale WebGoat em uma VM Ubuntu e use Kali para explorar as vulnerabilidades. Aprenda a identificar SQLi, XSS, CSRF do ponto de vista do atacante.
3. **Análise de binário com Ghidra/radare2:** Pegue um binário Linux simples e faça engenharia reversa. Entenda como programas C compilados aparecem em assembly.

---

### 17. Parrot OS

**Base:** Debian Stable | **Modelo:** Rolling | **Package Manager:** APT + dpkg

#### Resumo de Propósito
Parrot OS tem duas edições: **Security** (similar ao Kali) e **Home** (para uso geral, focada em privacidade). A edição Home é interessante para devs que querem privacidade + um ambiente Debian moderno.

**Para programação:** Parrot Home é um Debian rolling com foco em privacidade. Para estudo de segurança, Kali é mais conhecida e tem mais suporte. Para uso geral, as outras distros desta lista são melhores.

**Quando usar em VM:** Se você está estudando privacidade e anonimização (Tor, I2P, VPNs) do ponto de vista técnico, Parrot Home é um bom ambiente.

---

### 18. Gentoo Linux

**Base:** Original | **Modelo:** Rolling | **Package Manager:** Portage (emerge)

#### Resumo de Propósito
Gentoo é a distro onde você compila **tudo** do código fonte. Não há binários pré-compilados — você baixa os fontes e compila com as flags de otimização que você especifica.

Isso soa extremo, mas o aprendizado é extraordinário. O Portage (gerenciador de pacotes) usa um sistema de USE flags que permite customizar quais features são compiladas em cada pacote.

**Gentoo é a escola definitiva de como o Linux funciona em baixo nível.**

#### Para que tipo de programação é melhor
- **Programação de sistemas, kernel, toolchains:** Quando você compila o GCC, você entende o GCC. Quando você configura USE flags para o Python, você entende suas dependências.
- **Otimização de performance:** Você pode compilar com `-O3 -march=native` para seu processador específico, resultando em binários otimizados para seu hardware.
- **Entendimento profundo de dependências:** O Portage te obriga a entender o grafo de dependências de cada pacote.
- **Programação embedded:** A filosofia de "compile apenas o que precisa" é muito similar ao desenvolvimento embedded.

#### Vantagens para uso em VM
- **Aprendizado incomparável:** Uma instalação completa do Gentoo do zero te ensina mais sobre Linux do que um ano com qualquer outra distro.
- Levíssimo após instalação (você instala apenas o que precisa).

#### Desvantagens e armadilhas
- **Tempo de compilação:** Em VM, compilar o sistema completo pode levar horas. Para minimizar: ative `distcc` para usar todos os núcleos do host, ou use `binpkg` (pacotes binários pré-compilados para o Gentoo).
- **Curva de aprendizado extrema:** Não é para iniciantes. Mas para alguém com 6 meses de experiência Linux, tentar Gentoo em VM é um divisor de águas.
- **Manutenção contínua:** Gentoo exige atenção constante. Não é uma distro "instala e esquece".
- **Não é prático para desenvolvimento cotidiano:** Ninguém usa Gentoo como ambiente de desenvolvimento principal (exceto entusiastas). É uma escola, não uma ferramenta de trabalho.

#### Mini-projetos de estudo recomendados
1. **Instalação completa seguindo o Handbook:** O Gentoo Handbook é um dos melhores documentos de aprendizado Linux. Leva 2–3 dias, mas você aprende tudo.
2. **Compilar um kernel customizado:** Configure o kernel Linux manualmente (make menuconfig) para incluir apenas os drivers que a VM precisa. Compile. Boot. Isso te ensina mais sobre o kernel do que qualquer livro.
3. **USE flags exploration:** Instale o mesmo pacote com diferentes USE flags e compare os binários resultantes (tamanho, funcionalidades disponíveis).
4. **Cross-compilation:** Configure um ambiente de cross-compilation para ARM. Compile um programa x86 para rodar em Raspberry Pi.

---

### 19. Void Linux

**Base:** Original | **Modelo:** Rolling | **Package Manager:** XBPS

#### Resumo de Propósito
Void Linux é notável por ser completamente independente — não é baseado em Debian, Ubuntu, Arch ou Red Hat. Foi construído do zero com:
- **runit** como init system (ao invés de systemd)
- **musl libc ou glibc** (você escolhe na instalação)
- **XBPS** como gerenciador de pacotes (muito rápido)

Para um estudante de Linux, Void é fascinante porque expõe você a alternativas ao `systemd` — o init system mais controverso do Linux.

#### Para que tipo de programação é melhor
- **Programação de sistemas:** Entender runit vs systemd é conhecimento valioso sobre como sistemas Unix funcionam.
- **Desenvolvimento em C sem glibc:** A variante musl da Void é excelente para desenvolvimento de software portável.
- **Ambientes minimalistas de servidor:** runit é simples de entender e configurar.

#### Vantagens para uso em VM
- Muito leve e rápido.
- XBPS é um dos gerenciadores de pacotes mais rápidos em existência.
- Excelente para aprender sobre init systems alternativos.

#### Desvantagens e armadilhas
- **runit é diferente do systemd:** Comandos, estrutura de serviços, logging — tudo é diferente. Ótimo para aprendizado, mas requer adaptação.
- **Comunidade menor:** Menos recursos de suporte.

#### Mini-projetos de estudo recomendados
1. **runit vs systemd:** Configure o mesmo serviço (Nginx, por exemplo) tanto em Void (runit) quanto em Fedora (systemd). Compare a estrutura dos arquivos de serviço, o sistema de logging (svlogd vs journald), o processo de boot.
2. **XBPS packaging:** Crie um template XBPS para empacotar um programa simples. Compare com PKGBUILD (Arch), spec file (RPM) e debian/control (deb).

---

### 20. Clear Linux

**Base:** Original | **Modelo:** Rolling | **Package Manager:** swupd

#### Resumo de Propósito
Clear Linux é mantida pela Intel (sim, a Intel tem uma distro Linux) e é focada em performance máxima em hardware Intel. Usa técnicas de otimização agressivas: auto-FDO (feedback-directed optimization), patches de performance específicos para Intel, e um sistema de atualização diferencial único (`swupd`).

**Para hardware AMD:** Clear Linux é otimizada para Intel. Em um Ryzen AMD, você não vai ter os benefícios de performance máximos. O `swupd` também usa um modelo de bundle diferente de tudo que existe.

**Para estudo:** É interessante para entender técnicas de otimização de performance e um modelo alternativo de gerenciamento de pacotes. Mas não é prioridade no seu caso com AMD.

---

### 21. Ubuntu Server

**Base:** Ubuntu | **Modelo:** Fixed Release (LTS) | **Package Manager:** APT + dpkg

#### Resumo de Propósito
Ubuntu Server merece menção separada do Ubuntu Desktop porque o ambiente é radicalmente diferente: sem GUI por padrão, otimizado para servidor, com `cloud-init` pré-instalado, e muitas vezes instalado via imagens cloud (`.img`) ao invés do instalador tradicional.

#### Para que tipo de programação é melhor
- **Backend development:** Node.js, Python FastAPI, Go, Java Spring Boot — tudo funciona perfeitamente em Ubuntu Server.
- **DevOps:** Ubuntu Server é o target principal de tutoriais de Ansible, Terraform, Chef, Puppet.
- **Banco de dados:** PostgreSQL, MySQL, MongoDB, Redis — todos têm PPAs ou repositórios oficiais para Ubuntu.
- **Microserviços:** Simular múltiplos serviços em VMs Ubuntu Server separadas.

#### Vantagens para uso em VM
- **Imagens cloud ultra-otimizadas:** A imagem `ubuntu-22.04-server-cloudimg-amd64.img` tem ~500 MB e inicia em segundos. Você pode provisionar uma VM Ubuntu Server em menos de 1 minuto com cloud-init.
- **cloud-init nativo:** Configure IP, usuários, SSH keys, packages iniciais tudo via um arquivo YAML antes de ligar a VM.

```yaml
# Exemplo de cloud-init user-data
#cloud-config
package_update: true
packages:
  - nginx
  - python3-pip
  - postgresql
runcmd:
  - systemctl enable nginx
  - systemctl start nginx
```

#### Mini-projetos de estudo recomendados
1. **Vagrant + libvirt + Ubuntu Server:** Use Vagrant para provisionar VMs Ubuntu Server via código. Versione o `Vagrantfile` no Git.
2. **Ansible playbooks:** Escreva playbooks Ansible para configurar um servidor Ubuntu do zero. Aprenda idempotência.
3. **Docker Swarm multi-node:** Crie 3 VMs Ubuntu Server e configure um Docker Swarm. Deploy uma aplicação distribuída.

---

### 22. Fedora Server / Fedora CoreOS / Fedora Minimal

**Base:** Fedora | **Modelo:** Semi-rolling | **Package Manager:** DNF + RPM

#### Resumo de Propósito
Fedora tem várias edições que são valiosas para VMs de estudo:

- **Fedora Server:** Sem GUI, com Cockpit (painel de administração web) pré-instalado. Ótimo para simular servidores RHEL modernos.
- **Fedora CoreOS:** Distro imutável para containers. Configurada via Ignition (YAML), não tem `dnf`. Todo software roda em containers (Podman/Docker). Representa o futuro dos sistemas de servidor.
- **Fedora Minimal:** Instalação mínima sem GUI, sem Cockpit. O ponto de partida mais limpo possível.

#### Para que tipo de programação é melhor
- **CoreOS/Kubernetes:** Fedora CoreOS é o que roda nos nós de um cluster OpenShift (Kubernetes da Red Hat). Entender CoreOS é entender a plataforma onde microserviços enterprise rodam.
- **GitOps:** CoreOS é atualizado automaticamente via MicroOS concepts. A configuração é declarativa e imutável.

#### Mini-projetos de estudo recomendados
1. **Fedora CoreOS + Podman:** Crie uma VM CoreOS via Ignition config. Deploy sua aplicação como containers Podman. Explore o conceito de "container native infrastructure".
2. **Cockpit mastery:** Use Cockpit no Fedora Server para toda a administração. Instale o módulo Cockpit Machines para gerenciar VMs dentro da VM (nested virtualization).

---

### 23. Debian Testing / Sid

**Base:** Debian | **Modelo:** Rolling (Testing) / Instável (Sid) | **Package Manager:** APT + dpkg

#### Resumo de Propósito
Debian tem três branches além do Stable: **Testing** (próxima versão stable em preparação), **Unstable/Sid** (rolling release real), e **Experimental** (bleeding edge).

**Testing** é um ponto meio-termo interessante: pacotes mais recentes que o Stable, com um ciclo de testes que filtra problemas graves. É o que você usaria se quisesse um Debian mais atualizado mas não tão instável quanto o Sid.

#### Para estudo em VM
- **Debian Sid** é fascinante para entender como funciona o pipeline de empacotamento Debian. Você vê pacotes chegando do Experimental → Sid → Testing → Stable.
- Para contribuidores potenciais ao projeto Debian, entender o fluxo Sid/Testing é essencial.

---

### 24. Raspberry Pi OS (Raspbian)

**Base:** Debian | **Modelo:** Fixed Release | **Package Manager:** APT + dpkg

#### Resumo de Propósito
Raspberry Pi OS é a distro oficial do Raspberry Pi, baseada em Debian para arquitetura ARM. Em VM x86, você roda via emulação QEMU ARM — mais lento, mas totalmente funcional para aprendizado.

**Por que incluir em VMs x86?** Porque desenvolvimento para ARM/embedded é relevante para muitos contextos (IoT, microcontroladores, single-board computers). Simular Raspberry Pi OS em QEMU com emulação ARM te ensina cross-platform development.

#### Para que tipo de programação é melhor
- **IoT e sistemas embarcados em Python:** MicroPython patterns, GPIO simulation (via mock), MQTT, sensores.
- **ARM development:** Entender as diferenças entre x86 e ARM na perspectiva do desenvolvedor.
- **Programação em Python para projetos físicos.**

#### Configuração em VM
```bash
# Emulação QEMU ARM para Raspberry Pi OS
qemu-system-arm -M versatilepb \
  -kernel kernel-qemu-4.19.50-buster \
  -cpu arm1176 \
  -m 256 \
  -drive file=raspbian.img,format=raw
```

---

### 25. Windows 11 com WSL2

**Base:** Windows | **Modelo:** Fixed Release + Updates | **Package Manager:** winget, Chocolatey, Scoop + APT dentro do WSL2

#### Resumo de Propósito
Windows 11 com WSL2 (Windows Subsystem for Linux 2) é um caso especial: é um sistema Windows que roda um kernel Linux real em uma VM Hyper-V leve. Para um dev Linux, o interesse é entender como WSL2 funciona e o que ele significa para o desenvolvimento cross-platform.

**Para quem usa Fedora como host:** Você pode rodar Windows 11 em uma VM KVM no Fedora. Isso é útil para:
- Testar software que precisa de Windows
- Entender WSL2 "de dentro" (rodando Windows em VM que roda Linux)
- Desenvolver aplicações que precisam ser compatíveis com Windows

#### Para que tipo de programação é melhor
- **Desenvolvimento de aplicações cross-platform:** Testar que seu software Linux funciona também em Windows (via WSL2 ou nativamente).
- **Desenvolvimento .NET/C#:** O .NET Core roda em Linux, mas entender o ambiente original Windows é valioso.
- **Automação Windows com PowerShell:** Se você vai administrar ambientes Windows, PowerShell é essencial.
- **Desenvolvimento de jogos com Unity/Unreal:** Ambos rodam em Linux experimentalmente mas são muito mais estáveis em Windows.

#### Vantagens em VM KVM
- KVM + virtio drivers têm suporte no Windows (via driver pack da virtio-win).
- Windows 11 requer TPM 2.0, mas QEMU pode emular TPM via swtpm.

#### Configuração de TPM para Windows 11 em KVM
```bash
# Instalar swtpm
sudo dnf install swtpm swtpm-tools

# Criar estado TPM
mkdir -p /var/lib/libvirt/swtpm/windows11-vm
swtpm_setup --tpmstate /var/lib/libvirt/swtpm/windows11-vm \
  --tpm2 --create-ek-cert --create-platform-cert

# No virt-manager: Add Hardware → TPM → Model: TIS, Version: 2.0
# Backend: External → Path ao socket swtpm
```

#### Desvantagens e armadilhas
- **Pesado:** Windows 11 precisa de 4 GB de RAM mínimo para ser utilizável, 8 GB para ser confortável.
- **Licença:** Windows 11 requer licença paga (ou use as ISOs de avaliação de 90 dias da Microsoft).
- **Download grande:** ISO de ~6 GB.
- **VirtIO drivers:** Você precisa instalar os drivers VirtIO manualmente no Windows para ter performance adequada.

#### Mini-projetos de estudo recomendados
1. **WSL2 internals:** Instale WSL2 no Windows em VM. Explore `/proc/version` dentro do WSL2, a comunicação Windows ↔ Linux via 9P filesystem, e como os executáveis Windows podem ser chamados de dentro do WSL2.
2. **Cross-compilation Windows ↔ Linux:** Crie uma aplicação em Go que compile para ambos os alvos. Teste em ambos os ambientes.
3. **PowerShell scripting:** Aprenda os equivalentes PowerShell de comandos bash. Isso é valioso para qualquer sysadmin que precisa lidar com ambientes mistos.

---

## 4. Fedora no Fedora: Quando faz sentido? {#fedora-no-fedora}

Já que seu host é Fedora, há situações específicas onde rodar VMs Fedora faz mais sentido do que sair do ecossistema:

### Quando usar Fedora em VM
1. **Fedora Server para simular ambiente de servidor DNF/RPM:** Se você está desenvolvendo software que precisa ser testado em ambiente RHEL-like.
2. **Fedora CoreOS para containers:** Testar deployments containerizados no ambiente que a Red Hat usa para OpenShift.
3. **Fedora Minimal para entender o boot do zero:** Instalação mínima e construção camada a camada.
4. **Testar versões futuras do Fedora:** Seu host é Fedora 40? Rode Fedora 41 Rawhide em VM para testar incompatibilidades antes de atualizar.
5. **RPM packaging e teste:** Desenvolver pacotes RPM que precisam ser testados exatamente no mesmo ambiente do usuário final.

### Quando sair do ecossistema Fedora
1. **Para aprender apt/dpkg:** Use Ubuntu Server ou Debian. O mercado tem mais servidores baseados em Debian do que RHEL.
2. **Para aprender rolling release puro:** Use Arch ou EndeavourOS.
3. **Para aprender configuração declarativa:** Use NixOS.
4. **Para aprender sistemas alternativos (sem systemd):** Use Void Linux (runit) ou Alpine (OpenRC).
5. **Para simular ambientes de produção enterprise RHEL:** Use Rocky Linux (clone RHEL, mais estável que Fedora).
6. **Para aprender containers no nível mais fundamental:** Use Alpine Linux.

---

## 5. Plano de Estudos Sugerido {#plano}

Com base na sua situação (host Fedora, hardware AMD, 32 GB RAM, objetivos de desenvolvimento e estudo), aqui está um plano de estudos progressivo em 4 fases:

---

### FASE 1 — Base Sólida (Meses 1–2)
**Objetivo:** Dominar os dois ecossistemas fundamentais do mercado Linux.

#### VM 1: Ubuntu Server 24.04 LTS
**RAM:** 2 GB | **Disco:** 20 GB | **CPU:** 2 vCPUs

**Sequência de estudo:**
1. **Semana 1–2:** Instalação manual via ISO, configuração de rede, SSH hardening, criação de usuários, sudo, `ufw`.
2. **Semana 3–4:** Stack web completa: Nginx + Node.js (ou Python FastAPI) + PostgreSQL. Configure como serviços systemd. Aprenda `journalctl`, `systemctl`, `ss`, `netstat`.
3. **Semana 5–6:** apt em profundidade: PPAs, pinning, hold de pacotes, `apt-cache policy`. Instale múltiplas versões de Python com `pyenv`.
4. **Semana 7–8:** cloud-init: Destrua a VM e recrie usando uma imagem cloud + user-data YAML. Automatize toda a configuração inicial.

**Conhecimento adquirido:** apt/dpkg, systemd, UFW, cloud-init, serviços web em Linux, modelo de repositórios Debian/Ubuntu.

---

#### VM 2: Rocky Linux 9
**RAM:** 2 GB | **Disco:** 20 GB | **CPU:** 2 vCPUs

**Sequência de estudo:**
1. **Semana 1–2:** Instalação, comparação de estrutura com Ubuntu (onde os arquivos ficam, diferenças de localização de configs). `dnf` vs `apt`.
2. **Semana 3–4:** SELinux enforcing: instale um servidor web e faça-o funcionar **sem desabilitar SELinux**. Aprenda `setsebool`, `semanage`, `restorecon`, `audit2allow`.
3. **Semana 5–6:** `firewalld` com zonas. Compare com `ufw`. Configure zones para diferentes interfaces de rede.
4. **Semana 7–8:** AppStream modules: instale Python 3.11 e Node.js 20 em Rocky Linux 9 usando `dnf module`. Entenda o modelo de módulos do RHEL.

**Conhecimento adquirido:** dnf/RPM, SELinux, firewalld, AppStream modules, ecossistema RHEL/Red Hat.

---

### FASE 2 — Profundidade Técnica (Meses 3–4)
**Objetivo:** Entender Linux "por baixo" e o mundo rolling release.

#### VM 3: Arch Linux (instalação manual)
**RAM:** 1–2 GB | **Disco:** 20 GB | **CPU:** 2 vCPUs

**Sequência de estudo:**
1. **Semana 1:** Instalação **manual** seguindo o Arch Wiki. Sem archinstall script. Esta semana sozinha vai te ensinar mais sobre Linux do que qualquer outra.
2. **Semana 2–3:** pacman em profundidade: flags, hooks, `PKGBUILD` básico. AUR com `paru`. Entenda a diferença entre repositórios oficiais e AUR.
3. **Semana 4:** Rolling release na prática: acompanhe as atualizações por 2 semanas. Leia os anúncios de breaking changes. Aprenda a fazer downgrade com `pacman -U`.
4. **Semana 5–8:** Use Arch como ambiente de desenvolvimento principal (dentro da VM). Configure um ambiente completo para sua linguagem favorita usando apenas pacotes Arch e AUR.

**Conhecimento adquirido:** pacman/AUR, instalação manual Linux, rolling release management, PKGBUILD, Arch Wiki (que é útil para qualquer distro).

---

#### VM 4: Alpine Linux (versão minimal)
**RAM:** 512 MB | **Disco:** 5 GB | **CPU:** 1 vCPU

**Sequência de estudo:**
1. **Semana 1–2:** Instalação headless. Configure um servidor web básico. Entenda a diferença de musl libc vs glibc na prática.
2. **Semana 3–4:** Crie imagens Docker usando Alpine como base. Compare o tamanho e vulnerabilidades de uma imagem Ubuntu vs Alpine fazendo a mesma coisa.

**Conhecimento adquirido:** musl libc, minimalismo, containers Docker com imagens eficientes.

---

### FASE 3 — DevOps e Automação (Meses 5–6)
**Objetivo:** Simular ambientes de produção e praticar Infrastructure as Code.

#### VM 5: NixOS
**RAM:** 2 GB | **Disco:** 20 GB | **CPU:** 2 vCPUs

**Sequência de estudo:**
1. **Semana 1–2:** Instalação e entendimento do `configuration.nix`. Configure o sistema declarativamente.
2. **Semana 3–4:** Nix Flakes: crie environments reproduzíveis para projetos.
3. **Semana 5–8:** Home Manager: gerencie seu ambiente de usuário como código. Versione tudo no Git.

**Conhecimento adquirido:** Nix language, configuração declarativa, reprodutibilidade de ambientes.

---

#### Cluster: 3x Ubuntu Server + Vagrant + Ansible
**RAM:** 1 GB cada | **CPU:** 1 vCPU cada

**Sequência de estudo:**
1. Configure 3 VMs Ubuntu Server via Vagrant (Vagrantfile no Git).
2. Escreva playbooks Ansible para:
   - Configuração inicial (usuários, SSH, firewall)
   - Deploy de um load balancer (Nginx)
   - Deploy de dois backends (Node.js ou Python)
   - Deploy de um banco de dados (PostgreSQL)
3. Destrua as VMs e recrie apenas com `vagrant up` + `ansible-playbook`. Tudo deve funcionar automaticamente.

**Conhecimento adquirido:** Vagrant, Ansible, Infrastructure as Code, topologias de serviços.

---

### FASE 4 — Especialização (Meses 7–8)
**Objetivo:** Aprofundar em uma área específica baseado nos seus objetivos.

#### Se seu foco é Segurança:
- VM: Kali Linux + rede isolada com VM vulnerável
- Projetos: CTF challenges, OWASP Top 10 prático, análise de binários

#### Se seu foco é Ciência de Dados/IA:
- VM: Ubuntu Server 24.04 com CUDA (se tiver GPU dedicada) ou CPU-only
- Projetos: JupyterHub, MLflow, DVC, pipeline de ML reproduzível

#### Se seu foco é Kernel/Sistemas:
- VM: Gentoo Linux (compile o sistema)
- Projetos: Kernel module simples, compilação customizada do kernel, análise de syscalls com strace

#### Se seu foco é Cloud-Native/Kubernetes:
- VM: Fedora CoreOS + MicroK8s ou k3s
- Projetos: Deploy de aplicação em Kubernetes local, Helm charts, GitOps com ArgoCD

---

## 6. Tabela de Referência Rápida {#tabela}

| # | Distro | Base | Pkg Manager | Release Model | Prioridade | RAM VM | Foco Principal |
|---|--------|------|------------|--------------|-----------|--------|----------------|
| 1 | Ubuntu LTS | Debian | apt | Fixed | ⭐⭐⭐⭐⭐ | 512MB-4GB | Web, DevOps, IA |
| 2 | Debian Stable | Original | apt | Fixed | ⭐⭐⭐⭐ | 256MB-2GB | Servidor, Base |
| 3 | Arch Linux | Original | pacman | Rolling | ⭐⭐⭐⭐⭐ | 512MB-2GB | Dev avançado, Aprendizado |
| 4 | EndeavourOS | Arch | pacman | Rolling | ⭐⭐⭐ | 512MB-2GB | Arch acessível |
| 5 | Fedora WS | Original | dnf | Semi-rolling | ⭐⭐⭐⭐ | 2-4GB | RHEL upstream, Podman |
| 6 | CentOS Stream | RHEL | dnf | Rolling | ⭐⭐⭐ | 1-2GB | RHEL dev |
| 7 | Rocky Linux | RHEL | dnf | Fixed | ⭐⭐⭐⭐ | 1-2GB | Enterprise RHEL |
| 8 | AlmaLinux | RHEL | dnf | Fixed | ⭐⭐⭐ | 1-2GB | Enterprise RHEL |
| 9 | openSUSE Leap | SUSE | zypper | Fixed | ⭐⭐ | 1-2GB | SAP, YaST |
| 10 | openSUSE TW | SUSE | zypper | Rolling | ⭐⭐⭐ | 1-2GB | Rolling RPM |
| 11 | Linux Mint | Ubuntu | apt | Fixed | ⭐⭐ | 2-4GB | Desktop |
| 12 | Pop!_OS | Ubuntu | apt | Fixed | ⭐⭐ | 2-4GB | Desktop dev |
| 13 | Manjaro | Arch | pacman | Rolling | ⭐⭐ | 2-4GB | Arch fácil |
| 14 | NixOS | Original | nix | Fixed/Unstable | ⭐⭐⭐⭐ | 2GB | IaC, Reprodutibilidade |
| 15 | Alpine Linux | Original | apk | Fixed/Edge | ⭐⭐⭐⭐ | 128-512MB | Containers, Minimal |
| 16 | Kali Linux | Debian | apt | Rolling | ⭐⭐⭐ | 2-4GB | Segurança |
| 17 | Parrot OS | Debian | apt | Rolling | ⭐⭐ | 2-4GB | Segurança/Privacidade |
| 18 | Gentoo | Original | emerge | Rolling | ⭐⭐⭐ | 2-4GB | Kernel, Sistemas |
| 19 | Void Linux | Original | xbps | Rolling | ⭐⭐⭐ | 512MB-1GB | runit, sem systemd |
| 20 | Clear Linux | Original | swupd | Rolling | ⭐ | 2-4GB | Performance Intel |
| 21 | Ubuntu Server | Debian | apt | Fixed | ⭐⭐⭐⭐⭐ | 512MB-2GB | Backend, DevOps |
| 22 | Fedora Server/CoreOS | Original | dnf/Ignition | Semi-rolling | ⭐⭐⭐⭐ | 1-2GB | Containers, OpenShift |
| 23 | Debian Testing/Sid | Debian | apt | Rolling | ⭐⭐⭐ | 512MB-2GB | Empacotamento Debian |
| 24 | Raspberry Pi OS | Debian | apt | Fixed | ⭐⭐ | 256MB (QEMU ARM) | IoT, ARM |
| 25 | Windows 11 + WSL2 | Windows | winget+apt | Fixed | ⭐⭐⭐ | 4-8GB | Cross-platform |

**Legenda de Prioridade:**
- ⭐⭐⭐⭐⭐ = Alta prioridade para seus objetivos
- ⭐⭐⭐⭐ = Importante, inclua no plano
- ⭐⭐⭐ = Valioso para especialização específica
- ⭐⭐ = Opcional, explore se tiver interesse específico
- ⭐ = Baixa prioridade para AMD hardware

---

## Apêndice: Dicas Práticas de KVM/libvirt para AMD Ryzen 5825U

### Configuração recomendada do virt-manager para VMs de estudo

```xml
<!-- Configuração típica para VM de desenvolvimento -->
<domain type="kvm">
  <cpu mode="host-passthrough" check="none" migratable="on"/>
  <!-- host-passthrough expõe todos os features da CPU ao guest -->
  <!-- Melhor performance, mas a VM não pode migrar para outro host -->

  <features>
    <acpi/>
    <apic/>
  </features>

  <!-- Para VMs Linux: use VirtIO para tudo -->
  <devices>
    <disk type="file" device="disk">
      <driver name="qemu" type="qcow2" cache="none" io="native"/>
      <!-- cache="none" + io="native" = melhor performance de I/O -->
    </disk>

    <interface type="network">
      <model type="virtio"/>
      <!-- Sempre use virtio para rede, muito mais rápido que e1000 -->
    </interface>

    <video>
      <model type="virtio" heads="1" primary="yes"/>
      <!-- VirtIO GPU para melhor performance de display -->
    </video>

    <channel type="unix">
      <target type="virtio" name="org.qemu.guest_agent.0"/>
      <!-- QEMU Guest Agent: permite snapshots consistentes e ver IP da VM -->
    </channel>
  </devices>
</domain>
```

### Snapshots — seu melhor amigo para estudo

```bash
# Criar snapshot antes de um experimento perigoso
virsh snapshot-create-as --domain ubuntu-server \
  --name "antes-de-instalar-kubernetes" \
  --description "Estado limpo antes do lab K8s"

# Listar snapshots
virsh snapshot-list ubuntu-server

# Reverter para snapshot
virsh snapshot-revert ubuntu-server "antes-de-instalar-kubernetes"

# Deletar snapshot
virsh snapshot-delete ubuntu-server "antes-de-instalar-kubernetes"
```

### Rede para laboratórios isolados

```bash
# Criar uma rede isolada (sem acesso à internet) para labs de segurança
virsh net-define /dev/stdin << 'EOF'
<network>
  <name>lab-isolado</name>
  <bridge name="virbr-lab"/>
  <ip address="192.168.100.1" netmask="255.255.255.0">
    <dhcp>
      <range start="192.168.100.10" end="192.168.100.50"/>
    </dhcp>
  </ip>
  <!-- SEM <forward> = sem acesso externo -->
</network>
EOF

virsh net-start lab-isolado
virsh net-autostart lab-isolado
```

### Imagens cloud para provisionamento rápido

```bash
# Baixar imagem cloud Ubuntu Server (uma vez, reutilize)
wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img

# Criar disco a partir da imagem cloud (copy-on-write, não modifica o original)
qemu-img create -f qcow2 -F qcow2 \
  -b jammy-server-cloudimg-amd64.img \
  ubuntu-server-lab.qcow2 20G

# Criar cloud-init ISO
cat > user-data << 'EOF'
#cloud-config
hostname: ubuntu-lab
users:
  - name: dev
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa AAAA... (sua chave SSH pública)
packages:
  - git
  - curl
  - vim
  - htop
runcmd:
  - echo "VM pronta!" > /tmp/ready
EOF

cat > meta-data << 'EOF'
instance-id: ubuntu-lab-01
local-hostname: ubuntu-lab
EOF

genisoimage -output cloud-init.iso \
  -volid cidata -joliet -rock \
  user-data meta-data

# A VM inicia com tudo configurado automaticamente
```

---

*Documento gerado como guia de referência para estudo de distribuições Linux em VMs KVM/libvirt.*
*Hardware: AMD Ryzen 7 5825U | 32 GB RAM | Fedora Workstation como host*
*Última atualização: Abril 2026*
