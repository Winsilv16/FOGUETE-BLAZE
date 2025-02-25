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

nave_img = pygame.image.load("Foguete Blaze.png")
asteroide_img = pygame.image.load("asteroide_screen.png")
fundo_img = pygame.image.load("Espaçosideral_screen.jpg")
tiro_img = pygame.image.load("MIÍSSIL.gif") 

nave_img = pygame.transform.scale(nave_img, (50, 50))
asteroide_img = pygame.transform.scale(asteroide_img, (50, 50))
fundo_img = pygame.transform.scale(fundo_img, (largura, altura))
tiro_img = pygame.transform.scale(tiro_img, (10, 20))  


som_tiro = pygame.mixer.Sound("Som de Tiro de Sub Metralhadora UMP-45, barulhos de tiros - Efeitos Sonoros HD (mp3cut.net).mp3")
som_explosao = pygame.mixer.Sound("Atomic Explosion Meme (mp3cut.net).mp3")


pygame.mixer.music.load("BEAT_NO_TEMPO_DOS_IMBU_-_Nunca_vi_um_imbu_ser_tao_gostoso_-_Viral_(FUNK_REMIX)_by_Sr._Dart_[_YouConvert.net_].mp3")
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
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.image = asteroide_img
        self.rect = self.image.get_rect()
        self.rect.x = random.randrange(largura - self.rect.width)
        self.rect.y = random.randrange(-100, -40)
        self.speedy = random.randrange(1, 8)
        self.speedx = random.randrange(-3, 3)

    def update(self):
        self.rect.x += self.speedx
        self.rect.y += self.speedy
        if self.rect.top > altura + 10 or self.rect.left < -25 or self.rect.right > largura + 20:
            self.rect.x = random.randrange(largura - self.rect.width)
            self.rect.y = random.randrange(-100, -40)
            self.speedy = random.randrange(1, 8)


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
            self.kill()  # Remove o projétil quando ele sai da tela... def exibir_placar(surf, texto, tamanho, x, y):

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

while no_menu:
    tela.blit(fundo_img, (0, 0))
    desenhar_texto(tela, "Foguete Blaze", 64, branco, largura // 2, altura // 4)
    desenhar_texto(tela, "Aperte ESPAÇO para começar", 32, branco, largura // 2, altura // 2)
    desenhar_texto(tela, "Aperte ESC para sair", 32, branco, largura // 2, (altura // 2) + 50)
    pygame.display.flip()
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            quit()
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                no_menu = False
            if event.key == pygame.K_ESCAPE:
                pygame.quit()
                quit()

todos_sprites = pygame.sprite.Group()
asteroides = pygame.sprite.Group()
projeteis = pygame.sprite.Group()
nave = Nave()
todos_sprites.add(nave)

for i in range(8):
    a = Asteroide()
    todos_sprites.add(a)
    asteroides.add(a)

while running:
    if game_over:
        tela.fill(preto)
        desenhar_texto(tela, "GAME OVER", 64, vermelho, largura // 2, altura // 4)
        desenhar_texto(tela, "Pontuação: " + str(pontuacao), 32, branco, largura // 2, altura // 2)
        desenhar_texto(tela, "Recorde: " + str(recorde), 32, branco, largura // 2, altura // 2 + 50)
        desenhar_texto(tela, "Asteroides Destruídos: " + str(contador_asteroides.obter_total()), 32, branco, largura // 2, altura // 2 + 100)
        desenhar_texto(tela, "Pressione qualquer tecla para jogar novamente", 24, branco, largura // 2, altura * 3 // 4)
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
                    nave = Nave()
                    todos_sprites.add(nave)
                    for i in range(8):
                        a = Asteroide()
                        todos_sprites.add(a)
                        asteroides.add(a)
                    pontuacao = 0
                    asteroide_frequencia = 60  # Resetar a frequência
                    pygame.mixer.music.play(-1)  # Reiniciar a música
                    contador_asteroides.resetar()

                    
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
        if agora - ultimo_asteroide >= asteroide_frequencia and len(asteroides) < max_asteroides:
            ultimo_asteroide = agora
            a = Asteroide()
            todos_sprites.add(a)
            asteroides.add(a)

        todos_sprites.update()

        colisoes = pygame.sprite.spritecollide(nave, asteroides, False)
        if colisoes:
            game_over = True
            if pontuacao > recorde:
                recorde = pontuacao
                salvar_recorde(recorde)    

        # Verificar colisões entre projéteis e asteroides
        colisoes_projeteis = pygame.sprite.groupcollide(projeteis, asteroides, True, True)
        if colisoes_projeteis:
            pontuacao += len(colisoes_projeteis) * 10  # Aumenta a pontuação por cada asteroide atingido
            asteroide_frequencia *= 0.9  # Aumenta a frequência de aparecimento dos asteroides
            if asteroide_frequencia < 10:  # Limite para evitar que fique muito rápido
                asteroide_frequencia = 10
            som_explosao.play()
            contador_asteroides.incrementa()

        tela.blit(fundo_img, (0, 0))  
        todos_sprites.draw(tela)
        # exibe a pontuação no canto superior esquerdo
        exibir_placar(tela, "Pontuação: " + str(pontuacao), 24, largura // 4, 10)
        # exibe os asteroides destruídos no canto superior direito
        exibir_placar(tela, "Asteroides Destruídos: " + str(contador_asteroides.obter_total()), 24, largura * 3 // 4, 10)
        exibir_placar(tela, "Recorde: " + str(recorde), 24, largura // 2, 10) 
        pygame.display.flip()

        pontuacao += 1


pygame.quit()
