#include <stdio.h>
#include <SDL2/SDL.h>
#include <SDL2/SDL_image.h>
#include <SDL2_ttf.h>

typedef struct
{
  int   x, y;
  short life;
  char  *name;
} PLAYER;
typedef struct 
{
    float x, y;
    SDL_Texture *inimigoTexture;
}INIMIGO;
typedef struct
{
    PLAYER player;
    INIMIGO inimigo[2];
    SDL_Texture *currentTexture;
    SDL_Texture *playerDireita;
    SDL_Texture *playerEsquerda;
    SDL_Texture *playerCima;
    SDL_Texture *playerBaixo;
} EVERYTHING;

void CriandoPersonagem(SDL_Renderer *renderer, EVERYTHING *everything)
{
    SDL_Surface  *playerDireitaSurface  = NULL;
    SDL_Surface  *playerEsquerdaSurface = NULL;
    SDL_Surface  *playerCimaSurface     = NULL;
    SDL_Surface  *playerBaixoSurface    = NULL;
    SDL_Surface  *inimigoSurface        = NULL;
    int i = 0;      
    
    playerDireitaSurface = IMG_Load("/home/luis/Documentos/jo/soldado2.png"); //direita
    if(playerDireitaSurface  == NULL)
    {
        printf("ERROR IMAGE\n");
        SDL_Quit();
    }
    everything->playerDireita = SDL_CreateTextureFromSurface(renderer, playerDireitaSurface);
    everything->currentTexture = everything->playerDireita;
    SDL_FreeSurface(playerDireitaSurface);
    
    playerEsquerdaSurface = IMG_Load("/home/luis/Documentos/jo/soldado2A.png"); //esquerda
    if(playerEsquerdaSurface == NULL)
    {
        printf("ERROR IMAGE\n");
        SDL_Quit();
    }
    everything->playerEsquerda = SDL_CreateTextureFromSurface(renderer, playerEsquerdaSurface);
    SDL_FreeSurface(playerEsquerdaSurface);

    playerCimaSurface = IMG_Load("/home/luis/Documentos/jo/soldado2W.png"); //cima
    if(playerCimaSurface == NULL)
    {
        printf("ERROR IMAGE\n");
        SDL_Quit();
    }
    everything->playerCima = SDL_CreateTextureFromSurface(renderer, playerCimaSurface);
    SDL_FreeSurface(playerCimaSurface);
    
    playerBaixoSurface = IMG_Load("/home/luis/Documentos/jo/soldado2S.png"); //baixo
    if(playerBaixoSurface == NULL)
    {
        printf("ERROR IMAGE\n");
        SDL_Quit();
    }
    everything->playerBaixo = SDL_CreateTextureFromSurface(renderer, playerBaixoSurface);
    SDL_FreeSurface(playerBaixoSurface);   
    for(i = 0; i<2; i++)
    {
        inimigoSurface = IMG_Load("/home/luis/Documentos/jo/bandido.png");
        if(inimigoSurface == NULL)
        {
            printf("ERRRO IMAGE\n");
            SDL_Quit();
        }
        everything->inimigo[i].inimigoTexture = SDL_CreateTextureFromSurface(renderer, inimigoSurface);
        SDL_FreeSurface(inimigoSurface);
    }
}

int processEvents(SDL_Window *window, EVERYTHING *everything, SDL_Renderer *renderer)
{
  SDL_Event event;
  int done = 0;
   
  while(SDL_PollEvent(&event))
  {
        switch(event.type)
        {
            case SDL_WINDOWEVENT_CLOSE:
        {
            if(window)
            {
                SDL_DestroyWindow(window);
                window = NULL;
                done = 1;
            }
        }
        break;
            case SDL_KEYDOWN:
            {
                switch(event.key.keysym.sym)
                {
                  case SDLK_SPACE:
                       done = 1;
                  break;
                }
            }
            break;
            case SDL_QUIT:
                done = 1;
            break;
        }
  }
  const Uint8 *state = SDL_GetKeyboardState(NULL);
  if(state[SDL_SCANCODE_LEFT])
  {  everything->player.x-=3;
     everything->currentTexture = everything->playerEsquerda;
  }
  if(state[SDL_SCANCODE_RIGHT])
  {
     everything->player.x+=3;
     everything->currentTexture = everything->playerDireita;
  }
  if(state[SDL_SCANCODE_DOWN])
  {
    everything->player.y+=3;
    everything->currentTexture = everything->playerBaixo;
  }
  if(state[SDL_SCANCODE_UP])
  { 
    everything->player.y-=3;
    everything->currentTexture = everything->playerCima;
  }
  return done;
}

void segue(EVERYTHING *everything)
{
    for(int i = 0; i<2; i++)
    {
        if(everything->inimigo[i].x < everything->player.x)
            everything->inimigo[i].x +=1;
        if(everything->inimigo[i].x > everything->player.x)
            everything->inimigo[i].x -=1;
        if(everything->inimigo[i].y > everything->player.y)
            everything->inimigo[i].y -=1;
        if(everything->inimigo[i].y < everything->player.y)
            everything->inimigo[i].y +=1;
    }
}
void criandoRender(SDL_Renderer *renderer, EVERYTHING *everything)
{
    SDL_SetRenderDrawColor(renderer,10,10,10,255);// rgb alpha
       
    SDL_RenderClear(renderer);
       
    SDL_SetRenderDrawColor(renderer,255,255,255,255); //pinta de branco

       
    SDL_Rect player1Rect= {everything->player.x , everything->player.y, 100, 100};
    SDL_RenderCopy(renderer, everything->currentTexture, NULL, &player1Rect);
    for(int i = 0; i<2; i++)
    {
        SDL_Rect inimigoiRect = {everything->inimigo[i].x, everything->inimigo[i].y, 100, 100};
        SDL_RenderCopy(renderer, everything->inimigo[i].inimigoTexture, NULL, &inimigoiRect);
    }
    SDL_RenderPresent(renderer);
}

void windowLimit(EVERYTHING *everything)
{
    if(everything->player.y + 100 > 700)
        everything->player.y=700 - 100;
    if(everything->player.y < 0)
        everything->player.y= 0;
    if(everything->player.x + 100 > 1000)
        everything->player.x = 1000-100;
    if(everything->player.x < 0)
        everything->player.x = 0;
    for(int i = 0; i<2; i++)
    {
        if(everything->inimigo[i].y + 100 > 700)
            everything->inimigo[i].y=700 - 100;
        if(everything->inimigo[i].y < 0)
            everything->inimigo[i].y= 0;
        if(everything->inimigo[i].x + 100 > 1000)
            everything->inimigo[i].x = 1000-100;
        if(everything->inimigo[i].x < 0)
            everything->inimigo[i].x = 0;
    }
}

int showmenu(SDL_Surface* screen,TTF_Font* font)
{

    uint32 time;
    int x,y;
    const int NMMENU=2;
    const char* labels[NUMMENU] = {"Continue","Exit"};
    SDL_Surface* menus[MUMMENU];
    bool selected[MUMMENU]={0,0};
    SDL_Color color[2] = {{255,255,255},{255,00,0}};

    menus[0]=TTF_RenderText_solid(font,labels[0],color[0]);
    menus[1]=TTF_RenderText_solid(font,labels[1],color[0]);
    SDL_Rect pos[NUMMENU];
    pos[0].x=screen->clip_rect.w/2 - menus[0]->clip_rect.w/2;
    pos[0].y=screen->clip_rect.h/2 - menus[0]->clip_rect.h;
    pos[1].x=screen->clip_rect.w/2 - menus[0]->clip_rect.w/2;
    pos[1].y=screen->clip_rect.h/2 + menus[0]->clip_rect.h;

    SDL_FillRect(screen,&screen->clip_rect,SDL_MapRGB(screen->format,0x00,0x00,0x00));

    SDL_Event event;
    while(1)
    {
        time=SDL_GetTicks();
        while(SDL_PollEvent(&event))
        {
            switch(event.type)
            {
                case SDL_QUIT:
                    for(int i=0;i<NUMMENU;i++)
                    {
                        SDL_FreeSurface(menus[0]);
                    }
                    return 1;
                case SDL_MOUSEMOTION:
                    x=event.motion.x;
                    y=event.motion.y;
                    for(int i=0;i<NUMMENU;i++)
                    {
                        if(x>=pos[i].x && x<=pos[i].w && y>=pos[i].y && y<=pos[i].y+pos[i].h)
                        {
                            if(!selected[i])
                            {
                                selected[i]=1;
                                SDL_FreeSurface(menus[i])
                                menus[i]=TTF_RenderText_solid(font,labels[i],color[1]);   
                            }
                        }
                        else
                        {
                            if(slected[i])
                            {
                                selected[i]=0;
                                SDL_FreeSurface(menus[i])
                                menus[i]=TTF_RenderText_solid(font,labels[i],color[0]);   
                            }
                        }
                    }
                    break;
                case SDL_MOUSEBUTTONDOWN:
                    x=event.button.x;
                    y=event.button.y;
                    for(int i=0;i<NUMMENU;i++)
                    {
                        if(x>=pos[i].x && x<=pos[i].w && y>=pos[i].y && y<=pos[i].y+pos[i].h)
                        {
                            for(int j=0;i<NUMMENU;i++)
                            {
                                SDL_FreeSurface(menus[j]);
                            }
                            return i;
                        }
                    }
                    break;
                case SDL_KEYDOWN:
                    if(event.key.keysym==SDLK_ESCAPE)
                    {
                        for(int i=0;i<NUMMENU;i++)
                        {
                            SDL_FreeSurface(menus[i]);
                        }
                        return 0;   
                    }
            }
        }
        for(int i=0;i<NUMMENU;i++){
            SDL_BlitSurface(menus[i],NULL,screen,&pos[i]);
        }
        SDL_Flip(screen);
        if(1000/30>(SDL_GetTicks()-time))
            SDL_Delay(1000/30-(SDL_GetTicks()-time));
    }
}
/*
antes do While(running)
vc coloca

int i=showmenu(screen,font);
if(i==1)
    running=false;
   
    e no
    case SDL_ESCAPE:
   
    tambem coloca

int i=showmenu(screen,font);
if(i==1)
    running=false;
    break;
espero ter ajudado::
*/
int main(int argc, char const *argv[])
{
    EVERYTHING everything;
    SDL_Window   *window   = NULL;
    SDL_Renderer *renderer = NULL;
    SDL_Surface* screen    = NULL;
    TTF_Font* font         = NULL;
    
    SDL_Init(SDL_INIT_VIDEO);

    everything.player.x = 220;
    everything.player.y = 140;

    everything.inimigo[0].x = 0;
    everything.inimigo[0].y = 0;
    everything.inimigo[1].x = 1000;
    everything.inimigo[1].y = 700;
   
    showmenu(screen,font);
    window   = SDL_CreateWindow("gamewindow",SDL_WINDOWPOS_UNDEFINED, SDL_WINDOWPOS_UNDEFINED,1000,700,SDL_WINDOW_SHOWN);
    renderer = SDL_CreateRenderer(window, -1, SDL_RENDERER_ACCELERATED);
    CriandoPersonagem(renderer, &everything);
    int done = 0;

    while(!done)
    {   
        criandoRender(renderer, &everything);
        windowLimit(&everything);
        done = processEvents(window, &everything, renderer);
        segue(&everything);
        SDL_Delay(10);
    }
    
    SDL_DestroyWindow(window);
    SDL_DestroyRenderer(renderer);
    SDL_Quit();
    return 0;
}
