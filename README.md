#Comentários: Costumo alterar somente as coordenadas dos clicks para fazer testes em casa. No banco rodo o script de coordenadas do mouse, pois esse script não precisa de nenhuma biblioteca. // O Chat GPT é um grande aliado para criação dos códigos, só supervisionar o que e como ele está fazendo, no fim aprendemos por osmose... // Sempre usar o github para compartilhar arquivos .exe, linhas de código e arquivos de texto entre o Banco e o mundo externo. Fico a disposição para esclarecer qualquer dúvida. 51992240410


# Tutorial-Banribot
Instalar a versão mais recente de python: https://www.python.org/ftp/python/3.12.6/python-3.12.6-amd64.exe
Instalar o VSCode: https://code.visualstudio.com/sha/download?build=stable&os=win32-x64-user

#Instalar bibliotecas do python: pip install xxxxxx  (exemplo: pip install pyautogui)
Bibliotecas usadas: 
pyinstaller
pyautogui
pynput
time
sys

#No VSCode

File>New Text File
Plain Text>Python> Só programar

#Para rodar o script
F5(Sempre tem q salvar um arquivo antes de rodar o script pela primeira vez)>Python Debugger>Python File?: O script vai estar rodando.

Shit+F5: Para de rodar o script

#Para criar um executável dos scripts:

Sempre criar uma pasta para cada script>Copiar o endereço da pasta como texto> Abrir o CMD> Comandar: ''CD (endereço em texto)'' exemplo: ''CD C:\Users\36441\OneDrive\Área de Trabalho\Programa''
Comandar: ''pyinstaller --onefile (nome do arquivo.py)'' exemplo: pyinstaller --onefile pos6.py

Aguardar terminar de instalar e na pasta ''Dist'' vai ter um executavel do seu código. Agora qualquer computador pode rodar o programa sem ter as bibliotecas do python instaladas.


#SCRIPT PARA COORDENADAS DO MOUSE: 
import ctypes
import time


# Estrutura POINT para receber as coordenadas x e y
class POINT(ctypes.Structure):
    _fields_ = [("x", ctypes.c_long), ("y", ctypes.c_long)]

# Função para obter a posição atual do cursor
def get_mouse_position():
    pt = POINT()
    ctypes.windll.user32.GetCursorPos(ctypes.byref(pt))
    return pt.x, pt.y

# Função para obter o tamanho da tela principal
def get_screen_size():
    user32 = ctypes.windll.user32
    user32.SetProcessDPIAware()  # Para capturar o tamanho correto em sistemas com DPI diferente
    screen_width = user32.GetSystemMetrics(0)
    screen_height = user32.GetSystemMetrics(1)
    return screen_width, screen_height

# Função principal para mostrar as coordenadas do mouse e tamanho da tela
def mostrar_posicao_mouse():
    screen_width, screen_height = get_screen_size()
    print(f"Tamanho da tela: largura={screen_width}, altura={screen_height}")
    
    try:
        last_x, last_y = None, None  # Variáveis para armazenar a última posição
        while True:
            x, y = get_mouse_position()
            
            # Exibe a posição do mouse e verifica se está dentro da tela
            if (x != last_x or y != last_y):
                print(f"Posição atual do mouse: x={x}, y={y} (dentro da tela? {0 <= x <= screen_width and 0 <= y <= screen_height})", end="\r")
                last_x, last_y = x, y  # Atualiza a última posição

            time.sleep(0.5)
    except KeyboardInterrupt:
        print("\nExecução interrompida.")

# Executa a função
if __name__ == "__main__":
    mostrar_posicao_mouse()

#Banri Script:
import pyautogui as bot
from pynput import keyboard
import time
import sys

# Definição do intervalo de cores alvo em RGB
min_color = (235, 213, 210)
max_color = (240, 212, 212)

# Definição da tolerância para pequenas variações nos valores RGB
tolerance = 5

# Flag para controlar o encerramento do script
stop_script = False

# Lista de coordenadas a serem verificadas
coordinates = [
    (1790, 467),  # Aqui estava faltando uma vírgula
    (1790, 497),  # Aqui estava faltando uma vírgula
    (1790, 557),  # Aqui estava faltando uma vírgula
    (1790, 587),
    (1790, 614),
    (1790, 634),
    (1790, 674),
    (1790, 704),
    (1790, 734),
    (1790, 764),
    (1790, 794),
    (1790, 824)
]

# Função para verificar se o pixel está dentro do intervalo de cores especificado, com tolerância
def is_color_in_range_with_tolerance(pixel):
    for i in range(3):  # Verifica R, G e B individualmente
        if not (min_color[i] - tolerance <= pixel[i] <= max_color[i] + tolerance):
            return False
    return True

# Função para mover para baixo até encontrar um pixel que não esteja no intervalo de cores
def find_and_click(x, y):
    global stop_script
    while True:  # Loop para continuar movendo até encontrar um pixel fora do intervalo de cores
        if stop_script:  # Verifica se o flag foi ativado para parar o loop
            print("Script interrompido pela hotkey.")
            sys.exit()  # Encerra completamente o script

        pixel_color = bot.pixel(x, y)  # Captura a cor do pixel nas coordenadas (x, y)
        print(f"Cor no pixel ({x}, {y}): {pixel_color}")  # Debug: mostra a cor atual

        if is_color_in_range_with_tolerance(pixel_color):
            print(f"Cor no intervalo encontrada em ({x}, {y}). Movendo 30 pixels para baixo.")
            y += 30  # Movendo 30 pixels para baixo
            bot.sleep(0.2)  # Pausa para garantir o movimento antes de verificar novamente
        else:
            print(f"Cor fora do intervalo em ({x}, {y}). Clicando na posição atual.")
            bot.click(x=x, y=y)  # Clicar quando a cor estiver fora do intervalo
            break

# Função para tratar eventos de teclado (ESC para interromper)
def on_press(key):
    global stop_script
    try:
        if key == keyboard.Key.esc:  # Use ESC para interromper o script
            stop_script = True
    except Exception as e:
        print(f"Erro ao verificar a hotkey: {e}")

# Inicia o listener de teclado para monitorar a tecla ESC
listener = keyboard.Listener(on_press=on_press)
listener.start()

# Esperar um pouco antes de iniciar
bot.sleep(1)

# Realizar o primeiro clique apenas uma vez no início
bot.click(x=120, y=1054)
bot.sleep(1.5)

# Loop principal
while True:
    if stop_script:
        print("Script interrompido pela hotkey.")
        sys.exit()  # Encerra completamente o script

    # Verificar as posições uma a uma e clicar apenas em uma delas (onde a cor estiver fora do intervalo)
    for check_x, check_y in coordinates:
        if stop_script:
            print("Script interrompido pela hotkey.")
            sys.exit()  # Encerra completamente o script
        
        find_and_click(check_x, check_y)
        break  # Sai do loop após encontrar a primeira posição clicável

    # Continuar com os cliques adicionais
    bot.sleep(0.50)
    bot.click(x=888, y=622)

    if stop_script:
        print("Script interrompido pela hotkey.")
        sys.exit()  # Encerra completamente o script

    bot.sleep(3.50)
    bot.click(x=953, y=611)

    # Movimento do mouse para a posição (795, 35)
    bot.moveTo(x=795, y=35)
    print("Movendo o mouse para a posição (795, 35)")

    # Pequena pausa para evitar uso excessivo da CPU
    bot.sleep(1.0)
