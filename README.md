import pygame
import random

pygame.init()
pygame.mixer.init()

largura = 800
altura = 600
tela = pygame.display.set_mode((largura, altura))
pygame.display.set_caption("Foguete Blaze")

branco = (255, 255, 255)
preto = (0, 0, 0)
vermelho = (255, 0, 0)

# Carregar imagens
nave_img = pygame.image.load("Foguete Blaze.png")
asteroide_img = pygame.image.load("asteroide_screen.png")
fundo_img = pygame.image.load("Espaçosideral_screen.jpg")
tiro_img = pygame.image.load("MIÍSSIL.gif")
monstro_img = pygame.image.load("MONSTRO.png")  # Imagem do monstro

# Escalar imagens
nave_img = pygame.transform.scale(nave_img, (50, 50))
asteroide_img = pygame.transform.scale(asteroide_img, (50, 50))
fundo_img = pygame.transform.scale(fundo_img, (largura, altura))
tiro_img = pygame.transform.scale(tiro_img, (10, 20))
monstro_img = pygame.transform.scale(monstro_img, (80, 80))  # Escala do monstro

# Carregar sons
som_tiro = pygame.mixer.Sound(
    "Som de Tiro de Sub Metralhadora UMP-45, barulhos de tiros - Efeitos Sonoros HD (mp3cut.net).mp3"
)
som_explosao = pygame.mixer.Sound("Atomic Explosion Meme (mp3cut.net).mp3")

pygame.mixer.music.load(
    "BEAT_NO_TEMPO_DOS_IMBU_-_Nunca_vi_um_imbu_ser_tao_gostoso_-_Viral_(FUNK_REMIX)_by_Sr._Dart_[_YouConvert.net_].mp3"
)
pygame.mixer.music.play(-1)


class Nave(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.image = nave_img
        self.rect = self.image.get_rect()
        self.rect.centerx = largura // 2
        self.rect.bottom = altura - 10
        self.speedx = 0

    def update(self):
        self.speedx = 0
        keystate = pygame.key.get_pressed()
        if keystate[pygame.K_LEFT]:
            self.speedx = -5
        if keystate[pygame.K_RIGHT]:
            self.speedx = 5
        self.rect.x += self.speedx
        if self.rect.left < 0:
            self.rect.left = 0
        if self.rect.right > largura:
            self.rect.right = largura


class Asteroide(pygame.sprite.Sprite):
    def __init__(self, velocidade_base):
        pygame.sprite.Sprite.__init__(self)
        self.image = asteroide_img
        self.rect = self.image.get_rect()
        self.rect.x = random.randrange(largura - self.rect.width)
        self.rect.y = random.randrange(-100, -40)
        self.speedy = random.randrange(1, 8) * velocidade_base
        self.speedx = random.randrange(-3, 3) * velocidade_base

    def update(self):
        self.rect.x += self.speedx
        self.rect.y += self.speedy
        if (
            self.rect.top > altura + 10
            or self.rect.left < -25
            or self.rect.right > largura + 20
        ):
            self.rect.x = random.randrange(largura - self.rect.width)
            self.rect.y = random.randrange(-100, -40)
            self.speedy = random.randrange(1, 8)
            self.speedx = random.randrange(-3, 3)


class Projeteil(pygame.sprite.Sprite):
    def __init__(self, nave):
        pygame.sprite.Sprite.__init__(self)
        self.image = tiro_img  # Usa a imagem carregada
        self.rect = self.image.get_rect()
        self.rect.centerx = nave.rect.centerx
        self.rect.top = nave.rect.top
        self.speedy = -10  # Velocidade do projétil

    def update(self):
        self.rect.y += self.speedy
        if self.rect.bottom < 0:
            self.kill()  # Remove o projétil quando ele sai da tela


class Monstro(pygame.sprite.Sprite):
    def __init__(self, velocidade_base):
        pygame.sprite.Sprite.__init__(self)
        self.image = monstro_img
        self.rect = self.image.get_rect()
        self.rect.x = random.randrange(largura - self.rect.width)
        self.rect.y = random.randrange(-150, -80)
        self.speedy = random.randrange(1, 3) * velocidade_base  # Mais lento que asteroides
        self.speedx = random.choice([-2, 2]) * velocidade_base  # Movimento lateral
        self.health = 3  # Vida do monstro (quantos tiros aguenta)

    def update(self):
        self.rect.x += self.speedx
        self.rect.y += self.speedy

        # Reverter direção ao atingir as bordas
        if self.rect.left < 0 or self.rect.right > largura:
            self.speedx *= -1

        if self.rect.top > altura + 10:
            self.rect.x = random.randrange(largura - self.rect.width)
            self.rect.y = random.randrange(-150, -80)
            self.speedy = random.randrange(1, 3)
            self.speedx = random.choice([-2, 2])

    def tomar_dano(self):
        self.health -= 1
        if self.health <= 0:
            self.kill()  # Remove o monstro se a vida acabar
            return True  # Indica que o monstro foi destruído
        return False  # Indica que o monstro ainda está vivo


class ContadorAsteroides:
    def __init__(self):
        self.total_destruidos = 0

    def incrementa(self):
        self.total_destruidos += 1

    def resetar(self):
        self.total_destruidos = 0

    def obter_total(self):
        return self.total_destruidos


def exibir_placar(surf, texto, tamanho, x, y):
    fonte = pygame.font.Font(None, tamanho)
    texto_surface = fonte.render(texto, True, branco)
    texto_rect = texto_surface.get_rect()
    texto_rect.midtop = (x, y)
    surf.blit(texto_surface, texto_rect)


def desenhar_texto(surf, texto, tamanho, cor, x, y):
    fonte = pygame.font.Font(None, tamanho)
    texto_surface = fonte.render(texto, True, cor)
    texto_rect = texto_surface.get_rect()
    texto_rect.center = (x, y)
    surf.blit(texto_surface, texto_rect)


def carregar_recorde():
    try:
        with open("recorde.txt", "r") as f:
            return int(f.read())
    except FileNotFoundError:
        return 0


def salvar_recorde(recorde):
    with open("recorde.txt", "w") as f:
        f.write(str(recorde))


# Função para criar asteroides com base na dificuldade
def criar_asteroides(num_asteroides, velocidade_base):
    for i in range(num_asteroides):
        a = Asteroide(velocidade_base)
        todos_sprites.add(a)
        asteroides.add(a)


# Variáveis Globais
clock = pygame.time.Clock()
pontuacao = 0
recorde = carregar_recorde()
game_over = False
running = True
no_menu = True
asteroide_frequencia = 60  # Inicialmente, um asteroide a cada 60 frames
ultimo_asteroide = pygame.time.get_ticks()
max_asteroides = 20
contador_asteroides = ContadorAsteroides()
dificuldade = 1  # Nível de dificuldade padrão
monstros = pygame.sprite.Group()  # Grupo para os monstros
monstro_frequencia = 7000  # Um monstro a cada 7 segundos (em milissegundos)


# Menu de Dificuldade
def menu_dificuldade(game_state):
    dificuldade = game_state['dificuldade']
    asteroide_frequencia = game_state['asteroide_frequencia']
    max_asteroides = game_state['max_asteroides']
    monstro_frequencia = game_state['monstro_frequencia']

    selecionando_dificuldade = True
    while selecionando_dificuldade:
        tela.blit(fundo_img, (0, 0))
        desenhar_texto(
            tela, "Selecione a Dificuldade", 48, branco, largura // 2, altura // 4
        )
        desenhar_texto(
            tela, "1 - Fácil", 32, branco, largura // 2, altura // 2 - 30
        )
        desenhar_texto(
            tela, "2 - Médio", 32, branco, largura // 2, altura // 2 + 20
        )
        desenhar_texto(
            tela, "3 - Difícil", 32, branco, largura // 2, altura // 2 + 70
        )
        desenhar_texto(
            tela,
            "Pressione a tecla correspondente",
            24,
            branco,
            largura // 2,
            altura * 3 // 4,
        )
        pygame.display.flip()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                quit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_1:
                    game_state['dificuldade'] = 1
                    game_state['asteroide_frequencia'] = 60
                    game_state['max_asteroides'] = 10
                    game_state['monstro_frequencia'] = float("inf")
                    selecionando_dificuldade = False
                elif event.key == pygame.K_2:
                    game_state['dificuldade'] = 2
                    game_state['asteroide_frequencia'] = 45
                    game_state['max_asteroides'] = 15
                    game_state['monstro_frequencia'] = 7000
                    selecionando_dificuldade = False
                elif event.key == pygame.K_3:
                    game_state['dificuldade'] = 3
                    game_state['asteroide_frequencia'] = 30
                    game_state['max_asteroides'] = 20
                    game_state['monstro_frequencia'] = 5000
                    selecionando_dificuldade = False

    return game_state


# Menu Inicial
game_state = {
    'dificuldade': 1,
    'asteroide_frequencia': 60,
    'max_asteroides': 20,
    'monstro_frequencia': 7000,
    'ultimo_monstro': pygame.time.get_ticks()
}

while no_menu:
    tela.blit(fundo_img, (0, 0))
    desenhar_texto(tela, "Foguete Blaze", 64, branco, largura // 2, altura // 4)
    desenhar_texto(
        tela, "Aperte ESPAÇO para começar", 32, branco, largura // 2, altura // 2
    )
    desenhar_texto(
        tela, "Aperte ESC para sair", 32, branco, largura // 2, (altura // 2) + 50
    )
    pygame.display.flip()
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            quit()
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                no_menu = False
                game_state = menu_dificuldade(game_state)  # Abre o menu de dificuldade
            if event.key == pygame.K_ESCAPE:
                pygame.quit()
                quit()


# Inicialização do Jogo
todos_sprites = pygame.sprite.Group()
asteroides = pygame.sprite.Group()
projeteis = pygame.sprite.Group()
nave = Nave()
todos_sprites.add(nave)

# Ajustar a criação de asteroides com base na dificuldade
if game_state['dificuldade'] == 1:
    criar_asteroides(8, 0.5)  # Fácil: 8 asteroides, velocidade base 0.5
elif game_state['dificuldade'] == 2:
    criar_asteroides(12, 0.75)  # Médio: 12 asteroides, velocidade base 0.75
elif game_state['dificuldade'] == 3:
    criar_asteroides(16, 1)  # Difícil: 16 asteroides, velocidade base 1


# Loop Principal do Jogo
running = True
while running:
    if game_over:
        tela.fill(preto)
        desenhar_texto(tela, "GAME OVER", 64, vermelho, largura // 2, altura // 4)
        desenhar_texto(
            tela, "Pontuação: " + str(pontuacao), 32, branco, largura // 2, altura // 2
        )
        desenhar_texto(
            tela,
            "Recorde: " + str(recorde),
            32,
            branco,
            largura // 2,
            altura // 2 + 50,
        )
        desenhar_texto(
            tela,
            "Asteroides Destruídos: " + str(contador_asteroides.obter_total()),
            32,
            branco,
            largura // 2,
            altura // 2 + 100,
        )
        desenhar_texto(
            tela,
            "Pressione qualquer tecla para jogar novamente",
            24,
            branco,
            largura * 3 // 4,
        )
        pygame.display.flip()

        aguardando = True
        while aguardando:
            clock.tick(30)
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                    aguardando = False
                if event.type == pygame.KEYUP:
                    aguardando = False
                    game_over = False
                    # Reinicializar o jogo
                    todos_sprites = pygame.sprite.Group()
                    asteroides = pygame.sprite.Group()
                    projeteis = pygame.sprite.Group()
                    monstros = pygame.sprite.Group()  # Reinicializar o grupo de monstros
                    nave = Nave()
                    todos_sprites.add(nave)
                    pontuacao = 0
                    # Reset game state
                    game_state = {
                        'dificuldade': 1,
                        'asteroide_frequencia': 60,
                        'max_asteroides': 20,
                        'monstro_frequencia': 7000,
                        'ultimo_monstro': pygame.time.get_ticks()
                    }
                    game_state = menu_dificuldade(game_state)  # Abre o menu de dificuldade

                    # Recriar asteroides com base na dificuldade
                    if game_state['dificuldade'] == 1:
                        criar_asteroides(8, 0.5)
                    elif game_state['dificuldade'] == 2:
                        criar_asteroides(12, 0.75)
                    elif game_state['dificuldade'] == 3:
                        criar_asteroides(16, 1)

    else:
        clock.tick(60)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:  # Tecla de disparo
                    projeteil = Projeteil(nave)
                    todos_sprites.add(projeteil)
                    projeteis.add(projeteil)
                    som_tiro.play()

        # Verificar se é hora de criar um novo asteroide
        agora = pygame.time.get_ticks()
        if (
            agora - ultimo_asteroide >= game_state['asteroide_frequencia']
            and len(asteroides) < game_state['max_asteroides']
        ):
            ultimo_asteroide = agora
            a = Asteroide(game_state['dificuldade'] * 0.25 + 0.25)
            todos_sprites.add(a)
            asteroides.add(a)

        # Criar monstros periodicamente
        if game_state['dificuldade'] == 2:  # Monstros só aparecem no nível 2
            agora = pygame.time.get_ticks()
            if agora - game_state['ultimo_monstro'] >= game_state['monstro_frequencia']:
                game_state['ultimo_monstro'] = agora
                monstro = Monstro(game_state['dificuldade'] * 0.25)  # Velocidade baseada na dificuldade
                todos_sprites.add(monstro)
                monstros.add(monstro)

        todos_sprites.update()

        colisoes = pygame.sprite.spritecollide(nave, asteroides, False)
        if colisoes:
            game_over = True
            if pontuacao > recorde:
                recorde = pontuacao
                salvar_recorde(recorde)

        # Colisões de monstros com a nave
        colisoes_monstro_nave = pygame.sprite.spritecollide(nave, monstros, False)
        if colisoes_monstro_nave:
            game_over = True  # Fim de jogo se o monstro tocar a nave
            if pontuacao > recorde:
                recorde = pontuacao
                salvar_recorde(recorde)

        # Verificar colisões entre projéteis e asteroides
        colisoes_projeteis = pygame.sprite.groupcollide(
            projeteis, asteroides, True, True
        )
        if colisoes_projeteis:
            pontuacao += (
                len(colisoes_projeteis) * 10
            )  # Aumenta a pontuação por cada asteroide atingido
            game_state['asteroide_frequencia'] *= (
                0.9  # Aumenta a frequência de aparecimento dos asteroides
            )
            if game_state['asteroide_frequencia'] < 10:  # Limite para evitar que fique muito rápido
                game_state['asteroide_frequencia'] = 10
            som_explosao.play()
            contador_asteroides.incrementa()

        # Verificar colisões entre projéteis e monstros
        colisoes_projeteis_monstro = pygame.sprite.groupcollide(
            projeteis, monstros, True, False
        )  # Não remove o monstro imediatamente
        for projeteil, monstros_atingidos in colisoes_projeteis_monstro.items():
            for monstro in monstros_atingidos:
                if monstro.tomar_dano():  # Se o monstro for destruído
                    pontuacao += 50  # Dá mais pontos por destruir o monstro
                    som_explosao.play()

        tela.blit(fundo_img, (0, 0))
        todos_sprites.draw(tela)
        # exibe a pontuação no canto superior esquerdo
        exibir_placar(tela, "Pontuação: " + str(pontuacao), 24, largura // 4, 10)
        # exibe os asteroides destruídos no canto superior direito
        exibir_placar(
            tela,
            "Asteroides Destruídos: " + str(contador_asteroides.obter_total()),
            24,
            largura * 3 // 4,
            10,
        )
        exibir_placar(tela, "Recorde: " + str(recorde), 24, largura // 2, 10)
        pygame.display.flip()

        pontuacao += 1

pygame.quit()
