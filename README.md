# Orbito-Game-
% Orbito board game in Python with variable dimensions and bot that simulates the best play up to 2 moves ahead


visualiza([]):-!. % caso base da recursao do visualiza

visualiza([H|R]):- % escrever elemento por elemento atraves de recursao
    writeln(H),
    visualiza(R).

visualizaLinha(Lista):-
    visualizaLinhaAux(Lista, 1). % chamar de forma interativa um predicado auxiliar contendo um contador

visualizaLinhaAux([], _):-!. % caso base da recursao do prediacado auxiliar

visualizaLinhaAux([H|R], Contador):- % escrever elemento por elemento junto com o seu respectivo contador
    format("~w: ~w~n", [Contador, H]), % escrever a linha
    ContadorNew is Contador + 1,
    visualizaLinhaAux(R, ContadorNew).

insereObjecto((L,C), Tab, _):- % caso alguma das coor (L, C) nao esteja nas dimensoes de Tab, nao fazer nada
    length(Tab, NdeLinhas), % quantas linhas
    nth1(1, Tab, Linha1), length(Linha1, NdeColunas), % quantas colunas
    (not(between(1,NdeLinhas, L)); not(between(1, NdeColunas, C))) -> !; % verificar dimensoes
    nth1(L, Tab, LinhaTab), nth1(C, LinhaTab, El),
    (El == p ; El == e), !. % verificar se esta livre

insereObjecto((L,C), Tab, Obj):- % inserir o Obj na Tab nas coor (L, C)
    nth1(L, Tab, LinhaTab), nth1(C, LinhaTab, Obj).

insereVariosObjectos(ListaCoor, _, ListaObj):- % falhar caso ListaCoor e ListaObj nao sejam do mesmo tamanho
    length(ListaCoor, TamanhoListaCoor), length(ListaObj, TamanhoListaObj),
    TamanhoListaCoor =\= TamanhoListaObj, 
    !, fail.

insereVariosObjectos([], _, []):-!. % caso base da recursao de inserevariosobjetos

insereVariosObjectos([Coor | OutrasCoor], Tab, [Obj | OutrosObj]):- % se ListaCoor e ListaObj forem do mesmo tamanho, usar recusao para inserir elemento pro elemento 
    length(OutrasCoor, TamanhoOutrasCoor), length(OutrosObj, TamanhoOutrosObj),
    TamanhoOutrasCoor == TamanhoOutrosObj,
    insereObjecto(Coor, Tab, Obj),
    insereVariosObjectos(OutrasCoor, Tab, OutrosObj).

inserePontosVolta(Tab, (L, C)) :- % calcular as 8 posicoes à volta e adicionar pontos nelas caso sejam posicoes livres do Tab
    LinMenosUm is L - 1, LinMaisUm is L + 1,
    ColMenosUm is C - 1, ColMaisUm is C + 1,
    insereObjecto((LinMenosUm,ColMenosUm), Tab, p), insereObjecto((LinMenosUm,C), Tab, p), insereObjecto((LinMenosUm,ColMaisUm), Tab, p),
    insereObjecto((L,ColMenosUm), Tab, p), insereObjecto((L,ColMaisUm), Tab, p),
    insereObjecto((LinMaisUm,ColMenosUm), Tab, p), insereObjecto((LinMaisUm,C), Tab, p), insereObjecto((LinMaisUm,ColMaisUm), Tab, p).

inserePontos(_, []):-!. % caso base da recursao de inserePontos

inserePontos(Tab, [H|R]):- % inserir pontos elemento pro elemento da lista
    insereObjecto(H, Tab, p),
    inserePontos(Tab, R).

objectosEmCoordenadas([], _, []):- !. % base da recursao de objetosEmCoordenadas

objectosEmCoordenadas([(Lin, Col) | _], Tabuleiro, _):- % caso as corr (L, C) nao estajam nas dimansoes de Tab, falhar
    length(Tabuleiro, TamanhoTab),
    (not(between(1, TamanhoTab, Lin)); not(between(1, TamanhoTab, Col))),
    !, fail.

objectosEmCoordenadas([(Lin, Col) | RestoCoor], Tabuleiro, [Obj | RestoObj]):- % unificar recursivamente Obj de ListaObjs com o objeto encontrado em cada coor de ListaCoords, percorrendo as listas paralelamente
    nth1(Lin, Tabuleiro, Linha), nth1(Col, Linha, Elemento),
    Elemento = Obj,
    objectosEmCoordenadas(RestoCoor, Tabuleiro, RestoObj).

% Auxiliar que atraves das fbfs L, C, Tab e Obj é verdadeiro se nas coor (L, C) do Tab estiver o Obj
boolCoordElem((L, C), Tab, Obj):- % verifica se nas coor (L, C) de Tab tem o Obj
    nth1(L, Tab, Linha), nth1(C, Linha, Ele),
    Ele == Obj.

% Auxiliar que atraves das fbfs L, C e Tab é verdadeiro se nas coor (L, C) do Tab estiver uma variavel
coordenadaLivre((L, C), Tab):- % verifica se nas coor (L, C) tem uma variavel
    nth1(L, Tab, Linha), nth1(C, Linha, Ele),
    var(Ele).



coordObjectos(Objecto, Tabuleiro,  ListaCoords, ListaCoordObjs, NumObjectos):- % ListaCoordObjs é uma sublista de ListaCoords com apenas as coordenadas de ListaCoords que contem o Objecto e NumObjectos é o tamanho de ListaCoordObjs
    sort(ListaCoords, ListaCoordOrd), % ordenar a lista
    (
    (not(var(Objecto))) -> findall(Coor, (member(Coor, ListaCoordOrd), boolCoordElem(Coor, Tabuleiro, Objecto)), ListaCoordObjs) % se nao for uma variavel, usar o predicado auxiliar para filtrar todas as coor de ListaCoords que contenham esse Objecto
    ;
    findall(Coor, (member(Coor, ListaCoordOrd), coordenadaLivre(Coor, Tabuleiro)), ListaCoordObjs) % se for uma variavel, usar o predicado auxiliar para filtrar todas as coor de ListaCoords que contenham uma variavel
    ),
    length(ListaCoordObjs,NumObjectos). % calcular o tamanho de ListaCoorObjs

% Auxiliar  que atraves das fbfs Tab e ListaTodasCoor é verdadeiro se ListaTodasCoor for uma Lista com todas as coor do Tab
todasCoorTab(Tab, ListaTodasCoor):- % verifica as dimensoes de Tab e usa findall e between para criar a lista de todas as coor de Tab
    length(Tab, Dimensao),
    findall((L, C), (between(1,Dimensao, L), between(1,Dimensao,C)), ListaTodasCoor).

coordenadasVars(Tabuleiro, ListaVars):- % verifica quais das coor de Tab contem variaveis
    todasCoorTab(Tabuleiro, ListaTodasCoor),
    coordObjectos(_, Tabuleiro, ListaTodasCoor, ListaVarAux, _),
    ListaVars = ListaVarAux.

fechaListaCoordenadas(Tabuleiro, ListaCoord):- % se ListaCoord ja tiver 2 estrelas, encher o resto de ListaCoord com pontos
    coordObjectos(e, Tabuleiro, ListaCoord, LCO, 2), % verificar se tem 2 estrelas
    length(ListaCoord, TamanhoLista),
    QntsPontos is TamanhoLista - 2, 
    !,
    findall(p, between(1, QntsPontos, _), ListaPontos), % criar uma lista de pontos com o mesmo tamanho de ListaCoord - 2, ou seja, quantos pontos serao inseridos 
    findall(Coor, (member(Coor, ListaCoord), not(member(Coor, LCO))), ColocarPontos), % achar todas as coor que serao inseridos esses pontos
    insereVariosObjectos(ColocarPontos, Tabuleiro, ListaPontos).

fechaListaCoordenadas(Tabuleiro, ListaCoord):- % se ListaCoord ja tiver 1 estrelas e sobrar apenas 1 variavel, colocar uma estrela na variavel
    coordObjectos(e, Tabuleiro, ListaCoord, _, 1), % verificar se tem 1 estrela
    coordObjectos(_, Tabuleiro, ListaCoord, Vazio, 1), % verificar se tem 1 variavel apenas
    !,
    [Coor] = Vazio,
    insereObjecto(Coor, Tabuleiro, e), % inserir uma estrela na variavel
    inserePontosVolta(Tabuleiro, Coor). % inserir pontos à volta daquela estrela

fechaListaCoordenadas(Tabuleiro, ListaCoord):- % se ListaCoord tiver apenas 2 variaveis e nenhuma estrela, colocar uma estrela na variavel
    length(ListaCoord, TamanhoLista), 
    coordObjectos(p, Tabuleiro, ListaCoord, ListaPontos, NumPontos), 
    NumPontos =:= TamanhoLista - 2, % verificar se o numero de pontos de ListaCoord é igual ao TamanhoLista - 2, ou seja, se só há pontos e 2 variaveis na ListaCoord
    !,
    findall(Coor, (member(Coor, ListaCoord), not(member(Coor, ListaPontos))), ColocarEstrelas), % achar as coor das variaveis
    insereVariosObjectos(ColocarEstrelas, Tabuleiro, [e,e]), % inserir estrelas
    [Coor1, Coor2] = ColocarEstrelas,
    inserePontosVolta(Tabuleiro, Coor1), inserePontosVolta(Tabuleiro, Coor2). %  inserir pntos à volta das estrelas

fechaListaCoordenadas(_, _):-!. %  caso nao seja possivel fechar, nao gerar false

fecha(_, []):-!. % caso base da recusao de fecha

fecha(Tabuleiro, [Coordenadas | RestoCoordenadas]):- % fechar recursivamente uma por uma as listas de coor contidas em ListaListasCoord
    fechaListaCoordenadas(Tabuleiro, Coordenadas),
    fecha(Tabuleiro, RestoCoordenadas).


% Auxiliar que, atraves das fbfs Variaveis e ListaCoords, é verdadeira se os elementos de Variaveis estiverem em sequencia em ListaCoords
ehSequencia([_], _):-!. % caso base recursao ehSequencia

ehSequencia([Var1, Var2 | Variaveis], ListaCoords):- % verificar se os indices que estao em sequencia em Variaveis tambem estao em sequencia em ListaCoords
    nth1(IndexVar1, ListaCoords, Var1), nth1(IndexVar2, ListaCoords, Var2),
    IndexVar2 =:= IndexVar1 + 1,
    ehSequencia([Var2 | Variaveis], ListaCoords).


encontraSequencia(Tabuleiro, N, ListaCoords, Seq):- % Seq é uma lista de variveis de tamanho N da sequencia de variaveis de ListaCoords
    findall(Coor, (member(Coor, ListaCoords), coordenadaLivre(Coor, Tabuleiro)), Variaveis),
    length(Variaveis, TamanhoVariaveis),
    N == TamanhoVariaveis, % verificar se o numero de variaveis livres de ListaCoords é igual a N
    ehSequencia(Variaveis, ListaCoords), % verificar se sao seguidas
    append(Pre, Resto, ListaCoords),   
    append(Seq, Pos, Resto), % verificar se nao ha qualquer estrela em ListaCoords
    coordObjectos(e, Tabuleiro, Pre, _, 0), % verificar se nao ha qualquer estrela em ListaCoords antes de Seq
    coordObjectos(e, Tabuleiro, Pos, _, 0), % verificar se nao ha qualquer estrela em ListaCoords depois de Seq
    Variaveis = Seq, !.
    
aplicaPadraoI(Tabuleiro, [Coor1, Coor2, Coor3]):- % em caso de sequencia I aplicar o padrao da sequencia I 
    encontraSequencia(Tabuleiro, 3, [Coor1, Coor2, Coor3], Seq), 
    Seq = [Coor1, Coor2, Coor3], % verificar se Coor1, Coor2 e Coor3 sao uma sequencia
    insereObjecto(Coor1, Tabuleiro, e), insereObjecto(Coor3, Tabuleiro, e), %  colocar estrelas nas pontas
    inserePontosVolta(Tabuleiro, Coor1), inserePontosVolta(Tabuleiro, Coor3). %  colocar pontos á volta das estrelas

aplicaPadroes(_, []):-!. % caso base da recursao aplicaPadroes

aplicaPadroes(Tabuleiro, [ListaCoor | RestoListaListaCoords]) :- % se a sequencia de ListaCoor for de 3, aplicar o padrao I 
    encontraSequencia(Tabuleiro, 3, ListaCoor, Seq), 
    aplicaPadraoI(Tabuleiro, Seq), % aplicar padrao I
    aplicaPadroes(Tabuleiro, RestoListaListaCoords), !. % continuar a recursao

aplicaPadroes(Tabuleiro, [ListaCoor | RestoListaListaCoords]) :- % se a sequencia de ListaCoor for de 4, aplicar o padrao T
    encontraSequencia(Tabuleiro, 4, ListaCoor, Seq), 
    aplicaPadraoT(Tabuleiro, Seq), % aplicar padrao T
    aplicaPadroes(Tabuleiro, RestoListaListaCoords), !. % continuar a recursao

aplicaPadroes(Tabuleiro, [_ | RestoListaListaCoords]) :- % se a sequencia de ListaCoor nao for nem de 3 e nem de 4, nao aplicar padrao e seguir para o proximo
    aplicaPadroes(Tabuleiro, RestoListaListaCoords), !. % continuar a recursao

resolve(Estruturas, Tabuleiro):- % caso após a aplicaçao do aplicaPadroes e do fecha o tabuleiro e mantem igual, finalizar
    coordTodas(Estruturas, CT), % lista de todas as listas de colunas, linhas e areas
    coordenadasVars(Tabuleiro, Coord), % coordenadas das variaveis antes de alterar o tabuleiro
    aplicaPadroes(Tabuleiro, CT), 
    fecha(Tabuleiro, CT),
    coordenadasVars(Tabuleiro, Coord), !. % caso sejam as mesmas vasriaveis, parar e usar corte

resolve(Estruturas, Tabuleiro):- % caso após a aplicaçao do aplicaPadroes e do fecha o tabuleiro e mantem é diferente, dar sequencia ao loop
    coordTodas(Estruturas, CT), % lista de todas as listas de colunas, linhas e areas
    aplicaPadroes(Tabuleiro, CT), 
    fecha(Tabuleiro, CT),
    resolve(Estruturas, Tabuleiro). % caso não seja igual, aplicar a recursão