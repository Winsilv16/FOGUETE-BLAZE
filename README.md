# FOGUETE-BLAZE

import pygame
import random

pygame.init()

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

nave_img = pygame.transform.scale(nave_img, (50, 50))
asteroide_img = pygame.transform.scale(asteroide_img, (50, 50))
fundo_img = pygame.transform.scale(fundo_img, (largura, altura))

class Nave(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.image = nave_img 
        self.rect = self.image.get_rect()
        self.rect.centerx = largura // 2
        self.rect.bottom = altura - 10
        self.speedx = 0

