# pygame
import pygame
import sys
import random
from pygame.locals import *
WIDTH = 4
HEIGHT = 4
WINDOWWIDTH = 800
WINDOWHEIGHT = 600
FPS = 30
BLANK = None
BLACK = (0, 0, 0)
BLACK = (0, 0, 0)
GREEN = (0, 204, 0)
COLOR = (10,  50,  80)
WHITE = (255,255,255)

x1 = int((WINDOWWIDTH - (80 * WIDTH + (WIDTH - 1))) / 2)
y1 = int((WINDOWHEIGHT - (80 * HEIGHT + (HEIGHT - 1))) / 2)
x2 = int((WINDOWHEIGHT - (80 * HEIGHT + (HEIGHT - 1))) / 2)
y2 = int((WINDOWHEIGHT - (80 * HEIGHT + (HEIGHT - 1))) / 2)


def main():
    global FPSCLOCK, DISPLAYSURF, BASICFONT, RESET_SURF, RESET_RECT, NEW_SURF, NEW_RECT, SOLVE_SURF, SOLVE_RECT

    pygame.init()
    FPSCLOCK = pygame.time.Clock()
    DISPLAYSURF = pygame.display.set_mode((WINDOWWIDTH, WINDOWHEIGHT))
    pygame.display.set_caption('Slide Puzzle')
    BASICFONT = pygame.font.Font('freesansbold.ttf', 20)


    RESET_SURF, RESET_RECT = makeText('Reset',    (255, 255, 255), (0, 255, 0), WINDOWWIDTH - 120, WINDOWHEIGHT - 90)
    NEW_SURF,   NEW_RECT   = makeText('New Game', (255, 255, 255), (0, 255, 0), WINDOWWIDTH - 120, WINDOWHEIGHT - 60)
    SOLVE_SURF, SOLVE_RECT = makeText('Solve',    (255, 255, 255), (0, 255, 0), WINDOWWIDTH - 120, WINDOWHEIGHT - 30)

    mainBoard, solutionSeq = generateNewPuzzle(80)
    SOLVEDBOARD = getStartingBoard()
    allMoves = []

    while True:
        slideTo = None
        msg = 'Click tile or press arrow keys to slide.'
        if mainBoard == SOLVEDBOARD:
            msg = 'Solved!'

        drawBoard(mainBoard, msg)

        checkForQuit()
        for event in pygame.event.get():
            if event.type == MOUSEBUTTONUP:
                spotx, spoty = getSpotClicked(mainBoard, event.pos[0], event.pos[1])

                if (spotx, spoty) == (None, None):
                    if RESET_RECT.collidepoint(event.pos):
                        resetAnimation(mainBoard, allMoves)
                        allMoves = []
                    elif NEW_RECT.collidepoint(event.pos):
                        mainBoard, solutionSeq = generateNewPuzzle(80)
                        allMoves = []
                    elif SOLVE_RECT.collidepoint(event.pos):
                        resetAnimation(mainBoard, solutionSeq + allMoves)
                        allMoves = []
                else:

                    blankx, blanky = getBlankPosition(mainBoard)
                    if spotx == blankx + 1 and spoty == blanky:
                        slideTo = 'left'
                    elif spotx == blankx - 1 and spoty == blanky:
                        slideTo = 'right'
                    elif spotx == blankx and spoty == blanky + 1:
                        slideTo = 'up'
                    elif spotx == blankx and spoty == blanky - 1:
                        slideTo = 'down'

            elif event.type == KEYUP:
                if event.key in (K_LEFT, K_a) and isValidMove(mainBoard, 'left'):
                    slideTo = 'left'
                elif event.key in (K_RIGHT, K_d) and isValidMove(mainBoard, 'right'):
                    slideTo = 'right'
                elif event.key in (K_UP, K_w) and isValidMove(mainBoard, 'up'):
                    slideTo = 'up'
                elif event.key in (K_DOWN, K_s) and isValidMove(mainBoard, 'down'):
                    slideTo = 'down'

        if slideTo:
            slideAnimation(mainBoard, slideTo, 'Click tile or press arrow keys to slide.', 8)
            makeMove(mainBoard, slideTo)
            allMoves.append(slideTo)
        pygame.display.update()
        FPSCLOCK.tick(FPS)


def terminate():
    pygame.quit()
    sys.exit()


def checkForQuit():
    for event in pygame.event.get(QUIT):
        terminate()
    for event in pygame.event.get(KEYUP):
        if event.key == K_ESCAPE:
            terminate()
        pygame.event.post(event)


def getStartingBoard():
    counter = 1
    board = []
    for x in range(WIDTH):
        column = []
        for y in range(HEIGHT):
            column.append(counter)
            counter += WIDTH
        board.append(column)
        counter -= WIDTH * (HEIGHT - 1) + WIDTH - 1

    board[WIDTH-1][HEIGHT-1] = BLANK
    return board


def getBlankPosition(board):
    # Возвращаем x и y координат пустого места на доске.
    for x in range(WIDTH):
        for y in range(HEIGHT):
            if board[x][y] == BLANK:
                return (x, y)


def makeMove(board, move):
    # Эта функция проверяет верность хода.
    blankx, blanky = getBlankPosition(board)

#перемещение
    if move == 'up':
        board[blankx][blanky], board[blankx][blanky + 1] = board[blankx][blanky + 1], board[blankx][blanky]
    elif move == 'down':
        board[blankx][blanky], board[blankx][blanky - 1] = board[blankx][blanky - 1], board[blankx][blanky]
    elif move == 'left':
        board[blankx][blanky], board[blankx + 1][blanky] = board[blankx + 1][blanky], board[blankx][blanky]
    elif move == 'right':
        board[blankx][blanky], board[blankx - 1][blanky] = board[blankx - 1][blanky], board[blankx][blanky]


def isValidMove(board, move):
    blankx, blanky = getBlankPosition(board)
    return (move == 'up' and blanky != len(board[0]) - 1) or \
           (move == 'down' and blanky != 0) or \
           (move == 'left' and blankx != len(board) - 1) or \
           (move == 'right' and blankx != 0)


def getRandomMove(board, lastMove=None):
    # начало ходов переборе
    validMoves = ['up', 'down', 'left', 'right']

    if lastMove == 'up' or not isValidMove(board, 'down'):
        validMoves.remove('down')
    if lastMove == 'down' or not isValidMove(board, 'up'):
        validMoves.remove('up')
    if lastMove == 'left' or not isValidMove(board, 'right'):
        validMoves.remove('right')
    if lastMove == 'right' or not isValidMove(board, 'left'):
        validMoves.remove('left')

    return random.choice(validMoves)


def getLeftTopOfTile(tileX, tileY):
    left = x1 + (tileX * 80) + (tileX - 1)
    top = y1 + (tileY * 80) + (tileY - 1)
    return (left, top)


def getSpotClicked(board, x, y):
    # из координат x и y пикселей получить координаты x и y доски
    for tileX in range(len(board)):
        for tileY in range(len(board[0])):
            left, top = getLeftTopOfTile(tileX, tileY)
            tileRect = pygame.Rect(left, top, 80, 80)
            if tileRect.collidepoint(x, y):
                return (tileX, tileY)
    return (None, None)


def drawTile(tilex, tiley, number, adjx=0, adjy=0):
    left, top = getLeftTopOfTile(tilex, tiley)
    pygame.draw.rect(DISPLAYSURF, (0, 255, 0), (left + adjx, top + adjy, 80, 80))
    textSurf = BASICFONT.render(str(number), True, (255, 255, 255))
    textRect = textSurf.get_rect()
    textRect.center = left + int(80 / 2) + adjx, top + int(80 / 2) + adjy
    DISPLAYSURF.blit(textSurf, textRect)


def makeText(text, color, bgcolor, top, left):
    # создайте объекты Surface и Rect для некоторого текста и мыши
    textSurf = BASICFONT.render(text, True, color, bgcolor)
    textRect = textSurf.get_rect()
    textRect.topleft = (top, left)
    return (textSurf, textRect)


def drawBoard(board, message):
    DISPLAYSURF.fill(COLOR)
    if message:
        textSurf, textRect = makeText(message, (255, 255, 255), COLOR, 5, 5)
        DISPLAYSURF.blit(textSurf, textRect)

    for tilex in range(len(board)):
        for tiley in range(len(board[0])):
            if board[tilex][tiley]:
                drawTile(tilex, tiley, board[tilex][tiley])

    left, top = getLeftTopOfTile(0, 0)
    width = WIDTH * 80
    height = HEIGHT * 80
    pygame.draw.rect(DISPLAYSURF, (0, 50, 255), (left - 5, top - 5, width + 11, height + 11), 4)

    DISPLAYSURF.blit(RESET_SURF, RESET_RECT)
    DISPLAYSURF.blit(NEW_SURF, NEW_RECT)
    DISPLAYSURF.blit(SOLVE_SURF, SOLVE_RECT)


def slideAnimation(board, direction, message, animationSpeed):

    blankx, blanky = getBlankPosition(board)
    if direction == 'up':
        movex = blankx
        movey = blanky + 1
    elif direction == 'down':
        movex = blankx
        movey = blanky - 1
    elif direction == 'left':
        movex = blankx + 1
        movey = blanky
    elif direction == 'right':
        movex = blankx - 1
        movey = blanky

    drawBoard(board, message)
    baseSurf = DISPLAYSURF.copy()
    moveLeft, moveTop = getLeftTopOfTile(movex, movey)
    pygame.draw.rect(baseSurf, COLOR, (moveLeft, moveTop, 80, 80))


    for i in range(0, 80, animationSpeed):
        # анимирование плитки
        checkForQuit()
        DISPLAYSURF.blit(baseSurf, (0, 0))
        if direction == 'up':
            drawTile(movex, movey, board[movex][movey], 0, -i)
        if direction == 'down':
            drawTile(movex, movey, board[movex][movey], 0, i)
        if direction == 'left':
            drawTile(movex, movey, board[movex][movey], -i, 0)
        if direction == 'right':
            drawTile(movex, movey, board[movex][movey], i, 0)

        pygame.display.update()
        FPSCLOCK.tick(FPS)


def generateNewPuzzle(numSlides):
    # Из начальной конфигурации (1-15) сделать что-то новое
    # анимировать эти движения.

    sequence = []
    board = getStartingBoard()
    drawBoard(board, '')
    pygame.display.update()
    pygame.time.wait(500)
    lastMove = None
    for i in range(numSlides):
        move = getRandomMove(board, lastMove)
        slideAnimation(board, move, 'Generating new puzzle...', animationSpeed=int(80 / 3))
        makeMove(board, move)
        sequence.append(move)
        lastMove = move
    return (board, sequence)


def resetAnimation(board, allMoves):
    revAllMoves = allMoves[:]
    revAllMoves.reverse()

    for move in revAllMoves:
        if move == 'up':
            oppositeMove = 'down'
        elif move == 'down':
            oppositeMove = 'up'
        elif move == 'right':
            oppositeMove = 'left'
        elif move == 'left':
            oppositeMove = 'right'
        slideAnimation(board, oppositeMove, '', animationSpeed=int(80 / 2))
        makeMove(board, oppositeMove)


if __name__ == '__main__':
    main()
