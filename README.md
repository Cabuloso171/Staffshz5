-- BY NUKY - MENU DE INFORMAÇÕES HZ5
local imgui = require "mimgui"
local ffi = require "ffi"

local v = imgui.new.bool(false)
local input_text = imgui.new.char[256]()
local selected_category = imgui.new.int(-1)
local show_rules_window = imgui.new.bool(false)
local show_rules_categories = imgui.new.bool(false)
local show_jail_window = imgui.new.bool(false)
local show_jail_categories = imgui.new.bool(false)
local show_discord_window = imgui.new.bool(false)
local show_discord_categories = imgui.new.bool(false)
local show_search_window = imgui.new.bool(false)
local show_related_codes_window = imgui.new.bool(false)
local search_results = {}
local last_search_keyword = ""

local rainbow_colors = {
    imgui.ImVec4(1.0, 0.0, 0.0, 1.0),
    imgui.ImVec4(1.0, 0.5, 0.0, 1.0),
    imgui.ImVec4(1.0, 1.0, 0.0, 1.0),
    imgui.ImVec4(0.0, 1.0, 0.0, 1.0),
    imgui.ImVec4(0.0, 0.0, 1.0, 1.0),
    imgui.ImVec4(0.3, 0.0, 0.5, 1.0),
    imgui.ImVec4(0.5, 0.0, 0.5, 1.0)
}

local border_colors = {
    imgui.ImVec4(0.0, 0.7, 1.0, 1.0),
    imgui.ImVec4(1.0, 1.0, 1.0, 1.0)
}

local key_colors = {
    button = imgui.ImVec4(0.2, 0.2, 0.2, 1.0),
    text = imgui.ImVec4(1.0, 1.0, 1.0, 1.0),
    hover = imgui.ImVec4(0.3, 0.3, 0.3, 1.0),
    active = imgui.ImVec4(0.4, 0.4, 0.4, 1.0)
}

local key_sizes = {
    normal = imgui.ImVec2(40, 40),
    enter = imgui.ImVec2(80, 85),
    space = imgui.ImVec2(200, 40),
    backspace = imgui.ImVec2(60, 40)
}

local color_index = 1
local color_timer = 0
local color_change_speed = 0.01

local function get_current_rainbow_color()
    local next_index = color_index % #rainbow_colors + 1
    local progress = color_timer / color_change_speed
    
    local current_color = rainbow_colors[color_index]
    local next_color = rainbow_colors[next_index]
    
    return imgui.ImVec4(
        current_color.x + (next_color.x - current_color.x) * progress,
        current_color.y + (next_color.y - current_color.y) * progress,
        current_color.z + (next_color.z - current_color.z) * progress,
        1.0
    )
end

function render()
    color_timer = color_timer + imgui.GetIO().DeltaTime
    if color_timer >= color_change_speed then
        color_timer = 0
        color_index = color_index % #rainbow_colors + 1
    end
    
    local current_rainbow_color = get_current_rainbow_color()
    
    imgui.PushStyleColor(imgui.Col.Button, current_rainbow_color)
    imgui.PopStyleColor()
end

local keyboard_rows = {
    {"Q", "W", "E", "R", "T", "Y", "U", "I", "O", "P"},
    {"A", "S", "D", "F", "G", "H", "J", "K", "L"},
    {"Z", "X", "C", "V", "B", "N", "M"}
}

local bible_rp = {
    ["RP"] = "SIMULAR A VIDA REAL",
    ["ANT RP"] = "FAZER ALGO QUE NAO FARIA NA VIDA REAL | PUNICAO: 180 MINUTOS DE CADEIA",
    ["DM"] = "MATAR ALGUEM SEM MOTIVO | PUNICAO: 150 MINUTOS DE CADEIA",
    ["VDM"] = "USAR UM VEICULO COMO ARMA PRA MATAR ALGUEM | PUNICAO: 150 MINUTOS DE CADEIA",
    ["KOS"] = "MATAR ALGUEM SO POR SER DE UMA ORG RIVAL OU POR SER UM POLICIAL | PUNICAO: 150 MINUTOS DE CADEIA",
    ["MG"] = "MISTURAR OOC COM IC | PUNICAO: 180 MINUTOS DE CADEIA",
    ["IC"] = "DENTRO DO PERSONAGEM",
    ["OOC"] = "FORA DO PERSONAGEM",
    ["RK"] = "VOLTAR NA ACAO PRA SE VINGAR | PUNICAO: 200 MINUTOS DE CADEIA",
    ["DB"] = "ATIRAR DE DENTRO DO CARRO | PUNICAO: 300 MINUTOS DE CADEIA",
    ["PG"] = "FORCAR UMA ACAO | PUNICAO: 180 MINUTOS DE CADEIA (APENAS EM ACAO)",
    ["RPFTW"] = "FAZER RP DE SEMPRE GANHAR",
    ["PK"] = "MORTE DO PERSONAGEM",
    ["CK"] = "MORTE PERMANENTE DO PERSONAGEM",
    ["CB"] = "FICAR PROVOCANDO OS POLICIAIS PARA FORCAR UMA ACAO",
    ["CL"] = "SAIR DO GAME EM SITUACOES DE RP | PUNICAO: 1 DIA DE BAN",
    ["RDM"] = "MATAR PESSOAS ALEATORIAMENTE SEM MOTIVO | PUNICAO: 200 MINUTOS DE CADEIA (DMS CONSECUTIVOS)",
    ["NRA"] = "ESTAR EM LOCAL PUBLICO ARMADO SEM MOTIVO | PUNICAO: 50 MINUTOS DE CADEIA (SE ESTIVER MIRANDO)",
    ["HK"] = "MATAR USANDO HELISE DE HELICOPTERO",
    ["ASR"] = "AGREDIR SEM RAZAO | PUNICAO: 80 MINUTOS DE CADEIA (CORPO A CORPO)",
    ["SK"] = "MATAR NO SPAWN DE JOGADORES",
    ["IFP"] = "INFORMACAO DO PERSONAGEM",
    ["CMP"] = "CONTRA MAO PROPOSITAL",
    ["CJ"] = "ROUBAR VEICULO COM PESSOA DENTRO SEM RP",
    ["WH"] = "PULAR DE VEICULO EM ALTA VELOCIDADE",
    ["FD"] = "FLOODAR NO CHAT | PUNICAO: 50 MINUTOS DE CADEIA (APOS 3 VEZES)",
    ["AI"] = "ACAO IMPOSSIVEL",
    ["ATP"] = "ATRAPALHAR ACAO DE PROPOSITO",
    ["RT"] = "RELOGIO TRAVADO | PUNICAO: KICK",
    ["ZZ"] = "CORRER EM ZIG ZAG",
    ["CC"] = "CONTA COMPROMETIDA",
    ["NS"] = "NAO TER AMOR A VIDA | PUNICAO: 200 MINUTOS DE CADEIA",
    ["TK"] = "MATAR MEMBRO DA PROPRIA ORGANIZACAO | PUNICAO: 180 MINUTOS DE CADEIA",
    ["FF"] = "MATAR MEMBRO DA ORGANIZACAO ALIADA",
    ["MF"] = "FARMAR DINHEIRO OU ITENS E TRANSFERIR ENTRE CONTAS | PUNICAO: BAN PERMANENTE",
    ["MIX"] = "ABREVIAR PALAVRAS IC | PUNICAO: 180 MINUTOS DE CADEIA",
    ["BH"] = "CORRER PULANDO",
    ["DARK RP"] = "ROLEPLAY COM RACISMO, ASSERIO, XENOFOBIA, ETC - PROIBIDO | PUNICAO: 1 DIA DE BAN (CASOS EXTREMOS: BAN PERMANENTE)",
    ["AFK"] = "AUSENTE DO TECLADO DURANTE RP | PUNICAO: KICK (SE ESTIVER ATRAPALHANDO)",
    ["SURF"] = "FICAR EM CIMA DE VEICULO EM MOVIMENTO | PUNICAO: 10 MINUTOS DE CADEIA",
    ["MUC"] = "USAR CANAIS ERRADOS DE COMUNICACAO | PUNICAO: 80 MINUTOS DE CADEIA",
    ["DARKRP"] = "ROLEPLAY COM ASSERIO, RACISMO, SUICIDIO, ETC - PROIBIDO | PUNICAO: 1 DIA DE BAN (CASOS EXTREMOS: BAN PERMANENTE)",
    ["FP"] = "FINALIZAR PLAYER - NAO E PROIBIDO NO SERVIDOR"
}

local keyword_mappings = {
    ["arma"] = {"VDM", "NRA", "HK", "DB"},
    ["policia"] = {"KOS", "CB"},
    ["veiculo"] = {"VDM", "DB", "CJ", "WH", "SURF", "CMP"},
    ["player"] = {"DM", "RDM", "ASR", "SK", "FP", "PK", "CK"},
    ["morte"] = {"DM", "RDM", "KOS", "HK", "SK", "PK", "CK", "TK", "FF"},
    ["acao"] = {"ANT RP", "PG", "RK", "ATP", "AI", "CB", "CL", "NS"},
    ["chat"] = {"FD", "MUC", "MIX"},
    ["movimento"] = {"BH", "ZZ", "WH", "SURF"},
    ["organizacao"] = {"TK", "FF", "MF"},
    ["preconceito"] = {"DARK RP", "DARKRP"}
}

local server_rules = {
    ["UCP/Páginas/Regras"] = {
        "1. Proibido divulgar informações pessoais de outros jogadores",
        "2. Proibido qualquer tipo de discriminação ou preconceito",
        "3. Respeitar todos os membros da comunidade"
    },
    ["PROIBIÇÕES GERAIS"] = {
        "1. Proibido comércio externo, incluindo a venda de coins, dinheiro, veículos, casas, empresas, skins, acessórios ou contas.",
        "2. Proibido usar nicks ofensivos ou impróprios.",
        "3. Proibido o uso de contas secundárias para farmar bens, itens ou dinheiro. Transferências entre contas não são permitidas.",
        "4. Proibido ofender jogadores, a administração ou o servidor.",
        "5. Proibido qualquer tipo de discriminação, preconceito, homofobia, xenofobia ou atos que prejudiquem a imagem de alguém.",
        "6. Proibido divulgar outros servidores ou links externos.",
        "7. Proibido explorar bugs ou lags para obter vantagens.",
        "8. Proibido usar modificações ou trapaças que tragam benefícios indevidos.",
        "9. Proibido ofensas, calúnias, difamações e discriminações OOC.",
        "10. Proibido abordar (/ab) em locais safes, exceto policiais.",
        "11. Proibido o uso de sniper, exceto em territórios e invasões de favelas.",
        "12. Proibido spawn kill (matar jogadores ao entrar/sair de interiores ou favelas em abordagens).",
        "13. Proibido usar taser em trocas de tiros.",
        "14. Proibido fazer ação de patrulhamento solo, obrigatório a presença de dois ou mais policiais da mesma corporação.",
        "15. Proibido o uso de bombas (C4) com intuito de matar outro(s) player(s).",
        "16. Proibido o uso de mina terrestre em eventos.",
        "17. Proibido iniciar ação de loja com menos de 3 jogadores.",
        "18. Proibido iniciar qualquer tipo de ação/saquear contra policiais, mecânicos e samus fardados.",
        "19. Líderes e Sub-líderes têm obrigação de manter atividades dentro da cidade, caso ambos fiquem 3 dias ou mais ausentes a org/corp será resetada.",
        "20. Proibido correr para uma área safe/interior ou favela após ter sido iniciado uma ação fora dela (/ab ou trocação de tiro).",
        "21. Proibido a utilização de JBL em área safe.",
        "22. Proibido portar armas médias/pesadas, cometer crimes e/ou intervir em ações criminosas à paisana.",
        "23. Proibido apreender barraquinhas em áreas de desmanche/favela, exceto em dominação de favela.",
        "24. Proibido a cobrança de qualquer valor/item para efetuar o recrutamento de um player, seja corporação, organização, hospital e/ou mecânica.",
        "25. Proibido a utilização de motivos inválidos para (/algemar), obrigatório um motivo coerente.",
        "26. Líderes/sub-líderes de organização/corporação que infringirem alguma regra do servidor estarão gerando 1 (um) aviso para a organização/corporação na qual lidera.",
        "27. Membros de organização/corporação que forem banidos, irão gerar 1 (uma) advertência para a organização/corporação na qual participa.",
        "28. Líderes que forem banidos temporariamente ou permanentemente receberão 3 (três) avisos, sub-líderes receberão apenas 1 aviso.",
        "29. Proibido puxar veículos VIP durante ações.",
        "30. Para ações de TR, o tempo deverá ser respeitado, proibido atirar antes da contagem inicial terminar.",
        "31. Para ações de Casa roubável, poderão iniciar trocação somente no momento que der o anúncio da casa no chat global."
    },
    ["CHAT"] = {
        "1. Proibido flood e spam.",
        "2. Proibido usar a OLX para assuntos que não envolvem vendas legais.",
        "3. Proibido usar comandos de anúncios para conversar."
    },
    ["VOIP"] = {
        "1. Proibido tocar música no VOIP, exceto em locais reservados (Casa, Empresa e/ou festas particulares).",
        "2. Proibido usar o VOIP enquanto estiver ferido."
    },
    ["TERRITÓRIOS"] = {
        "1. Proibido participar de territórios fora da gangzone.",
        "2. Proibido retornar aos territórios após ser hospitalizado.",
        "3. O mínimo para iniciar um território é de 3 jogadores."
    },
    ["BANCO"] = {
        "1. Proibido matar o refém antes da negociação.",
        "2. O mínimo para iniciar um assalto ao banco é de 5 jogadores."
    },
    ["SEQUESTROS"] = {
        "1. Proibido sequestrar sem iniciar a ação (/ab).",
        "2. O mínimo para iniciar um sequestro é de 3 jogadores.",
        "3. A integridade do refém SEMPRE deve ser priorizada.",
        "4. Proibido bater no refém algemado, torturas, afogamento ou qualquer tipo de humilhação desnecessária.",
        "5. Durante o sequestro, o refém deverá ser alimentado e suprido (água, comida e medicamentos).",
        "6. Caso o refém desmaie de fome ou sede, deverá ser reanimado ou encaminhado ao hospital.",
        "7. Caso seja notado que o refém não avisou sobre a necessidade de se alimentar propositalmente para sair da ação, o mesmo poderá ser punido.",
        "8. Caso o refém seja muito exigente ou debochado quanto a isso, será encarado como NS.",
        "9. Os sequestradores são obrigados a informar em quantos estão, assim como a quantidade de reféns e o local da negociação via chamado para a polícia.",
        "10. Proibido o uso de helicópteros.",
        "11. É limitado o número de reféns por sequestro: os sequestradores deverão manter até 6 reféns no máximo.",
        "12. Tanto os sequestradores como os policiais devem priorizar a vida de seus próprios membros e dos reféns.",
        "13. Toda negociação passa a ser área safe. Caso os bandidos efetuem disparos durante a negociação do refém, a ação se transforma em uma ação de rua (pode ser considerado Anti RP).",
        "14. É proibida a entrada de novos participantes depois da ação já iniciada ou participantes que foram mortos voltarem à ação.",
        "15. É proibido quebrar a negociação de ambas as partes (punição: Anti RP).",
        "16. É permitido chamar o SAMU ao final da ação.",
        "17. Se um indivíduo morrer por algum motivo e um SAMU aparecer, o mesmo poderá efetuar seu trabalho sem ser atrapalhado nem sofrer nenhum atentado ou dano.",
        "18. O indivíduo que for reanimado por SAMU não poderá permanecer no local, a ação para ele se encerrou.",
        "19. É proibido o uso de qualquer área da dependência de qualquer criminalidade (favela, territórios dominados, desmanches, mercado negro, etc.).",
        "20. Os sequestradores poderão pedir armas, drogas e dinheiro no caso de organizações (negociável); a policiais, apenas dinheiro e fuga limpa; a civis, ao próprio sequestrado e/ou familiares, apenas 3% do valor total que o sequestrado tiver em sua conta."
    },
    ["ASSALTO A REFÉM (/AB)"] = {
        "1. Proibido reagir ao assalto antes de 10 segundos da abordagem.",
        "2. Proibido exigir dinheiro do banco ou itens que a vítima não possua no momento."
    },
    ["DESMANCHE"] = {
        "1. Proibido abordar (/ab) em desmanches, exceto quando o local estiver sob dominação de território, ações de caixa ou casa roubável.",
        "2. No desmanche 3, é considerado safe zone toda a área coberta por território, sendo proibido: segurar armas, abordagens, apreensão de barraquinha, e qualquer ação de combate ou roubo que não seja Casa roubável, caixa eletrônico, e TR.",
        "3. Fora de ação de TR, o território do desmanche 3 deverá ser safe, estando dominado ou pacificado."
    },
    ["CONTA"] = {
        "1. A conta é pessoal e intransferível. Caso seja punida ou banida, o servidor não se responsabiliza por seu uso."
    },
    ["ÁREAS SAFES"] = {
        "1. Áreas seguras incluem: hospitais, oficinas mecânicas, HQs de profissões, praças, fábricas de materiais/drogas, hotéis (apenas interior), concessionárias, delegacias, desmanches e bancos (exceto durante assaltos).",
        "2. Interiores de casas e empresas não são áreas seguras."
    },
    ["INVASÃO DE FAVELA"] = {
        "1. O mínimo para iniciar uma invasão de favela é de 5 jogadores.",
        "2. A invasão deve ser marcada com 15 minutos de antecedência e registrada no Discord com uma print e um dos motivos válidos, como anúncios de vendas na favela alvo pelo chat global."
    },
    ["REGRAS ADICIONAIS DO ROLEPLAY"] = {
        "1. Manter o roleplay em todas as situações.",
        "2. Proibido quebrar o roleplay.",
        "3. Seguir as regras gerais de roleplay.",
        "4. As regras do servidor devem prevalecer e serem seguidas acima de tudo.",
        "5. As regras de organizações e corporações devem ser seguidas, mas as exceções descritas podem alterar temporariamente essas regras para melhor desempenho do RP.",
        "6. Toda regra postada no Discord em abas específicas é considerada oficial.",
        "7. O Horizonte Roleplay 2 prioriza o Roleplay, não o PVP. Jogadores devem seguir lógica, história e objetivos, e ir contra isso pode resultar em punição.",
        "8. Para comprar uma corporação, o jogador deve abrir um ticket no Discord oficial para avaliação de maturidade e responsabilidade."
    },
    ["REGRAS DE ORGANIZAÇÕES E CORPORAÇÕES"] = {
        "1. A partir do 10º dia da criação, a organização ou corporação deve ter no mínimo 10 membros ativos.",
        "2. Se no 11º dia não atingir 10 membros ativos, será aplicada uma advertência e multa administrativa.",
        "3. Acumulação de 10 advertências consecutivas pode levar à reinicialização da organização/corporação.",
        "4. Para renovação a cada 30 dias, a organização/corporação deve ter mais de 15 membros ativos.",
        "5. Membros que se conectam esporadicamente (mais de 3 dias off) ou não participam não são considerados ativos.",
        "6. Organizações/corporações com advertência ativa não podem participar de eventos especiais até regularização."
    },
    ["CORREGEDORIA DE JUSTIÇA E TRIBUNAL FEDERAL"] = {
        "1. A Corregedoria investiga denúncias, irregularidades, avalia conduta de servidores públicos e fiscaliza princípios constitucionais.",
        "2. Pode interferir em ações policiais, prefeitura e corporações, composta por líderes, sub-líderes, prefeito, vice-prefeito e um presidente (juiz).",
        "3. Punições possíveis: afastamento do membro + blacklist de uma semana, prisão (staff) + afastamento, redução de salário, advertência, PD de 1 a 15 dias (banimento temporário, autorizado pela direção).",
        "4. Membro setado na corporação não pode sair antes de 7 dias, ou resultará em banimento de 5 dias.",
        "5. Membros afastados temporariamente não podem ser recrutados por outras corporações antes de 7 dias (blacklist temporária).",
        "6. Recrutamento só após análise da ficha criminal junto ao responsável de Org e Corp, com o indivíduo fora de ações criminosas por 5 dias.",
        "7. Proibida troca de nome sem abrir ticket, só após renovação e skin a cada 10 dias.",
        "8. Proibido usar skin de outra corporação, salvo permissão da corregedoria.",
        "9. Proibido usar skin diferente da corporação em PTR com outras guarnições.",
        "10. Proibido desacato entre líderes/membros ou conversas paralelas pelo /QRR.",
        "11. Proibido oficial fazer ações criminosas (desmanche, ações de família VIP, caixinha, assalto, sequestro, carro forte, roubo ao banco).",
        "12. Pode fazer parte de família, desde que não faça missões ilegais.",
        "13. Proibido usar tag de família VIP em serviço.",
        "14. Proibido usar mochila de dinheiro e móveis roubados como acessórios em serviço.",
        "15. Proibido prender policiais em serviço, à paisana ou com estrelas, salvo se estiverem com produtos ilegais.",
        "16. Oficial à paisana pode portar combat shotgun e desert para defesa; outras armas podem levar à condução e prisão.",
        "17. Proibido mal uso de armas (Taser, M4, MP5) para brincadeiras com oficiais.",
        "18. Proibido usar VTRs (QSV) sem uniforme da corporação.",
        "19. Proibido prender/injetar player que outra corporação já tenha feito flagrante e algemado.",
        "20. Proibido desacatar ou desobedecer um oficial da corregedoria.",
        "21. Proibido investigação não autorizada pela corregedoria.",
        "22. Proibido corrupção como vendas de armas, capacetes e coletes (permitido se for do VIP).",
        "23. Proibido desacatar cidadão abordado e fazer revista sem avisar.",
        "24. Proibido revistar player do sexo oposto; na ausência de mesmo sexo, pedir apenas RG.",
        "25. Em caso de mochila de dinheiro, policial pode pedir que o cidadão retire o acessório para provar que é acessório; descumprimento é motivo de prisão.",
        "26. Proibido atos de preconceito, ofensas, deboche ou qualquer ato que contrarie a ética profissional do policial."
    },
    ["CONFEDERAÇÃO CRIMINOSA E TRIBUNAL DO CRIME"] = {
        "1. Favelas podem se unir para ações, desde que ambas estejam de acordo e cumpram o RP.",
        "2. Existe BlackList para organizações: líder pode solicitar que outras orgs não recrutem um membro expulso por 7 dias, com motivo grave, explicação e provas claras.",
        "3. BlackList válida a partir da postagem, se aceita pelo responsável de orgs/corps.",
        "4. Líder que descumprir a BlackList receberá punição em sua org."
    },
    ["REGRAS GERAIS DE DENÚNCIAS NO SITE"] = {
        "1. A denúncia deve ser feita pelo próprio player ou por alguém que participou da ação em questão.",
        "2. Caso punido por denúncia no site, o jogador tem 48 horas para contestar; após isso, não há direito a reclamações ou revisão.",
        "3. Após denúncia feita/recebida, há 2 horas para postar contra-provas; após isso, a equipe decidirá (aceita/recusada).",
        "4. Proibido desrespeitar ou ofender um player durante uma denúncia.",
        "5. Denunciante deve garantir que denuncia a pessoa correta com provas suficientes; falta de provas pode levar a punição por falsa denúncia.",
        "6. Em caso de vídeo, verificar se foi enviado; se houver quebra de link, abrir nova denúncia ou ticket com o link.",
        "7. Provas adicionais após denúncia aceita/recusada serão rejeitadas, mesmo comprovando inocência/incriminação.",
        "8. Denunciante que cometer delito em sua prova poderá ser punido.",
        "9. Descumprimento de regras, Anti RP, uso de mods proibidos, etc., será punido, mesmo que a denúncia seja recusada.",
        "10. Para denúncia ser válida, deve conter: I. Provas (vídeo); II. RG do acusado, data, logo e chat do servidor; III. Motivo e breve descrição; IV. Provas dentro de 24 horas; V. Sem edições/cortes; VI. Sem provas de celular/prints/gravações sobrepostas; VII. Sem provas de outros jogadores; VIII. Vídeos com no máximo 5 minutos; IX. Sem fotos inadequadas; X. Prints permitidos como prova, com data, hora e informações de conta.",
        "11. Membros de org/corp com denúncia aceita geram 1 advertência para a org/corp.",
        "12. Líderes/Sub-líderes com denúncia aceita geram 1 aviso para a org/corp.",
        "13. Denúncias de CHEATER devem conter vídeo sem edições, com áudio original, TAB mostrando ping e ID do acusado.",
        "14. Denúncias de CL devem conter vídeo do momento do quit, mostrando o player quitando; veículo VIP sumindo não é válido; denunciante não precisa postar 1 minuto adicional, mas acusado deve provar retorno em 1 minuto se alegar crash.",
        "15. Denúncias com links não-YouTube ou de sites terceiros serão recusadas."
    }
}

local discord_rules = {
    ["REGRAS GERAIS DO DISCORD"] = {
        "1. Comércio externo resultará em banimento, tanto no Discord quanto in-game.",
        "2. Proibido qualquer forma de preconceito ou atos como assédio, abuso, racismo, homofobia e preconceitos (raça, cor, religião, bullying).",
        "3. Proibido exibir links com conteúdo impróprio (+18, susto, gore, hack, etc.).",
        "4. Proibida a divulgação de outros servidores ou links diretos.",
        "5. Proibido flood; uso desnecessário de @ será penalizado.",
        "6. Nicknames impróprios (pornografia, palavrões) são proibidos; 10 minutos para troca após advertência.",
        "7. Proibido marcar cargos ou staff sem motivo plausível.",
        "8. Proibido usar o VOIP enquanto estiver ferido.",
        "9. Proibido ruídos constantes ou perturbadores em canais de áudio; evite músicas ou sons desagradáveis.",
        "10. Proibido debater assuntos administrativos nos canais; abra ticket ou denuncie.",
        "11. Proibido divulgações no bate-papo; use a aba 'OLX'.",
        "12. Prazo para revisão de banimento: 7 dias.",
        "13. Prazo para revisão de cadeia staff: 48 horas.",
        "14. Prazo para revisão de denúncia aceita/recusada: 48 horas.",
        "15. Canais de comunicação (bate-papo, chat-vip, memes, design) não são para tirar dúvidas; use #duvidas."
    },
    ["TICKETS"] = {
        "1. Horário de atendimento: 09:00 às 00:00; após esse horário, a resposta pode variar.",
        "2. Seja objetivo ao abrir ticket; evite discussões ou brincadeiras; forneça informações claras.",
        "3. Tempo de espera após resposta do suporte: 20 minutos; sem interação, o ticket pode ser finalizado.",
        "4. Após análise de provas, não serão aceitas discussões adicionais; o ticket será finalizado.",
        "5. Punições do bot (Automod) não serão retiradas se o conteúdo for inaceitável.",
        "6. Revisão de punição via ticket deve ter prova concreta; reabrir o mesmo caso por inconformidade resultará em penalidade.",
        "7. Provas de punições podem ser editadas/censuradas por conter informações sigilosas; jogador pode exigir avaliação por responsável ou direção.",
        "8. Banimentos pelo guardião não mostram provas ao jogador; direito de avaliação por superior é válido."
    }
}

local jail_punishments = {
    ["10 MINUTOS"] = {
        "- Surf",
        "- ASM (Novatos)"
    },
    ["50 MINUTOS"] = {
        "- FLOOD (Apos 3)",
        "- NRA (Fazendo o ato de mirar)"
    },
    ["80 MINUTOS"] = {
        "- ASM (Agredindo sem motivo corpo a corpo)",
        "- LOJA SOLO/DUO",
        "- VIOLACAO DA BIBLIA RP",
        "- MUC Atendimento",
        "- MUC Duvida",
        "- MUC Missa",
        "- MUC NEWS",
        "- MUC OLX",
        "- MUC /Reportar",
        "- MUC /ANORG",
        "- MUC /AN (seja conversa ou farpa)",
        "(OBS: TODOS OS MUCS PODERAO SER RELEVADOS A DEPENDER DO MOTIVO)",
        "- PTR SOLO"
    },
    ["100 MINUTOS"] = {
        "- ABUSO DE TAB"
    },
    ["150 MINUTOS"] = {
        "- DM (Ferir / Matar sem motivo)",
        "- VDM (Ferir / Matar usando veiculo)",
        "- KOS"
    },
    ["180 MINUTOS"] = {
        "- Acao Safe",
        "- Acao Desmanche (fora de TR / ACAO DE CX / CASA)",
        "- AB Desmanche",
        "- Anti-RP",
        "- MG",
        "- MIX",
        "- PG (Apenas em Acao)",
        "- TK"
    },
    ["200 MINUTOS"] = {
        "- INVASAO SEM MARCAR",
        "- NS (SEM AMOR A VIDA)",
        "- RDM (DMS CONSECUTIVOS)",
        "- RK"
    },
    ["300 MINUTOS"] = {
        "- Corrupcao (se n fizer parte do RP)",
        "- DB"
    },
    ["KICKS"] = {
        "- RT / BUGADO / LAGGADO",
        "- BUGANDO / ATRAPALHANDO EVENTO (STAFF)",
        "- AFK (Apenas se estiver atrapalhando RP e/ou no meio da pista)",
        "- TROCA NICK (se voltar, o admin banira)",
        "- SKIN DO CJ (se voltar, admin banira o IP)"
    },
    ["BANIMENTOS TEMPORÁRIOS"] = {
        "1 DIA",
        "- Dark RP",
        "- CL",
        "- Injuria, Difamacao ou deboche com incitacao a rebelião contra staffs",
        "",
        "5 dias",
        "- Handlling",
        "- Animacao vantajosa",
        "- Anti RP extremo",
        "",
        "15 dias",
        "- Qbug, Cbug, Ebug, etc...",
        "- Demais cortes de Animacao",
        "- Abuso de bugs em eventos",
        "",
        "De 10 a 20 dias",
        "- Ma conduta"
    },
    ["BANIMENTOS PERMANENTES"] = {
        "(se e mod e da beneficio = Cheater)",
        "- Cheater no Geral (Mods Proibidos em geral)",
        "- Abuso de Bugs",
        "- Comercio Ilegal",
        "- Divulgacao de outros servidores",
        "- Ofensa ao Servidor",
        "- DARK RP EXTREMO (homofobia, preconceito, assedio, ETC...)",
        "- Nick Improprio",
        "- Skin do CJ apos primeiro kick (BAN APENAS IP)",
        "- Money Farm"
    },
    ["Tabela da censura (Mute Temporário de comunicação)"] = {
        "1- O mute nao censura o chat whatsapp IC",
        "2- O mute so sera aplicado se identificado que foi dito algo com intuito de ofender e ou debochar saindo do RP. (chingamentos feitos por chat vip serao interpretados como ofensa)",
        "3- Farpas comuns e pequenas ofensas (Dentro do RP) como: fraco, tapete, corno, gado, doido, idiota, imbecil, etc... sao permitidas",
        "4- O Mute pode ser acompanhado de outras punicoes, seja cadeias e/ou banimentos",
        "5- Se algo nao estiver na tabela ou nao se encaixar em nenhum dos topicos abaixo, sera avaliado e aplicado entre 2 e 10 dias",
        "",
        "1 DIA",
        "- Farpa media (com intuito de ofender: arrombado, retardado, lixo)",
        "- Utilizacao de nomes comparativos e apelidos: (bulliyng) - se a pessoa nao gostar de um apelido e o mesmo insistir",
        "- flood (apos prisao)",
        "- Palavreados de Baixo calao como ofensa: (EX: fdp, fds, tmnc; etc...)",
        "",
        "5 DIAS",
        "- Palavreados preconceituosos com deficiencia: (EX: tem down, Teleton, tijolinho)",
        "- Utilizacao de partes sexuais, partes intimas, e/ou palavriados vulgar obscenos (mesmo se for giria 'acalma a ppk' por ex)",
        "",
        "10 Dias",
        "- Palavreados Preconceituosos, ofensas, ou deboche com Doencas Terminais.",
        "",
        "Qualquer vocabulario mais pesado, com intuito de assediar ou conter preconceito sera motivo de banimento + calar de 30 dias"
    }
}

local color_index = 1
local border_color_index = 1
local anim_time = 0
local border_anim_time = 0
local title_pos = 0
local moving_right = true

local function append_to_input(text)
    local current_text = ffi.string(input_text)
    local new_text = current_text .. text
    if #new_text < 256 then
        ffi.copy(input_text, new_text)
    end
end

local function backspace_input()
    local current_text = ffi.string(input_text)
    if #current_text > 0 then
        ffi.copy(input_text, current_text:sub(1, -2))
    end
end

local function process_input()
    local text = ffi.string(input_text):upper()
    if bible_rp[text] then
        sampAddChatMessage(bible_rp[text], 0xFFFF0000)
    else
        search_results = {}
        last_search_keyword = text:lower()
        for code, desc in pairs(bible_rp) do
            if desc:lower():find(last_search_keyword) then
                table.insert(search_results, {code = code, desc = desc})
            end
        end
        if #search_results > 0 then
            show_search_window[0] = true
        else
            sampAddChatMessage("Nenhum resultado encontrado para '" .. text .. "'!", 0xFFFF)
        end
    end
    ffi.copy(input_text, "")
end

local function update_animations(delta_time)
    anim_time = anim_time + delta_time
    border_anim_time = border_anim_time + delta_time

    if anim_time > 0.15 then
        anim_time = 0
        color_index = color_index % #rainbow_colors + 1
    end

    if border_anim_time > 0.4 then
        border_anim_time = 0
        border_color_index = border_color_index % 2 + 1
    end
end

local function update_title_position(delta_time, window_width, text_width)
    if moving_right then
        title_pos = title_pos + 80 * delta_time
        if title_pos > (window_width - text_width - 30) then
            moving_right = false
        end
    else
        title_pos = title_pos - 80 * delta_time
        if title_pos < 30 then
            moving_right = true
        end
    end
end

local function draw_title()
    local window_width = imgui.GetWindowWidth()
    local text = "CHECAR INFO by NukY"
    local text_width = imgui.CalcTextSize(text).x

    imgui.SetCursorPosY(20)
    imgui.SetCursorPosX(title_pos)
    imgui.PushStyleColor(imgui.Col.Text, rainbow_colors[color_index])
    imgui.Text(text)
    imgui.PopStyleColor()

    update_title_position(imgui.GetIO().DeltaTime, window_width, text_width)
end

local function draw_input_field()
    imgui.SetCursorPosY(60)
    imgui.Separator()
    imgui.SetCursorPosY(80)
    imgui.TextColored(imgui.ImVec4(0.8, 0.8, 0.8, 1.0), "DIGITE A OPCAO ABAIXO:)")
    imgui.SetCursorPosY(110)
    imgui.PushItemWidth(550)
    if imgui.InputTextWithHint("##input", "Ex: BH ou 'arma'", input_text, 256, imgui.InputTextFlags.EnterReturnsTrue) then
        process_input()
    end
    imgui.PopItemWidth()
end

local function draw_rules_button()
    imgui.SetCursorPosY(150)
    if imgui.Button("REGRAS HZ5", imgui.ImVec2(180, 30)) then
        show_rules_categories[0] = not show_rules_categories[0]
    end
    imgui.SameLine()
    if imgui.Button("CADEIAS HZ5", imgui.ImVec2(180, 30)) then
        show_jail_categories[0] = not show_jail_categories[0]
    end
    imgui.SameLine()
    if imgui.Button("DISCORD HZ5", imgui.ImVec2(180, 30)) then
        show_discord_categories[0] = not show_discord_categories[0]
    end
end

local function draw_keyboard()
    local start_x = 30
    local start_y = imgui.GetCursorPosY() + 50

    imgui.PushStyleColor(imgui.Col.Button, key_colors.button)
    imgui.PushStyleColor(imgui.Col.ButtonHovered, key_colors.hover)
    imgui.PushStyleColor(imgui.Col.ButtonActive, key_colors.active)
    imgui.PushStyleColor(imgui.Col.Text, key_colors.text)

    imgui.SetMouseCursor(imgui.MouseCursor.Arrow)

    imgui.SetCursorPos(imgui.ImVec2(start_x, start_y))
    for _, key in ipairs(keyboard_rows[1]) do
        if imgui.Button(key, key_sizes.normal) then
            append_to_input(key)
            imgui.SetMouseCursor(imgui.MouseCursor.Arrow)
        end
        imgui.SameLine()
    end

    start_y = start_y + 45
    imgui.SetCursorPos(imgui.ImVec2(start_x + 20, start_y))
    for _, key in ipairs(keyboard_rows[2]) do
        if imgui.Button(key, key_sizes.normal) then
            append_to_input(key)
            imgui.SetMouseCursor(imgui.MouseCursor.Arrow)
        end
        imgui.SameLine()
    end

    start_y = start_y + 45
    imgui.SetCursorPos(imgui.ImVec2(start_x + 40, start_y))
    for i = 1, 7 do
        local key = keyboard_rows[3][i]
        if imgui.Button(key, key_sizes.normal) then
            append_to_input(key)
            imgui.SetMouseCursor(imgui.MouseCursor.Arrow)
        end
        imgui.SameLine()
    end

    imgui.SetCursorPos(imgui.ImVec2(start_x + 40 + 7 * 45, start_y))
    if imgui.Button("Enter", key_sizes.enter) then
        process_input()
        imgui.SetMouseCursor(imgui.MouseCursor.Arrow)
    end

    start_y = start_y + 50
    imgui.SetCursorPos(imgui.ImVec2((imgui.GetWindowWidth() - key_sizes.space.x) * 0.5, start_y))
    if imgui.Button("Espaco", key_sizes.space) then
        append_to_input(" ")
        imgui.SetMouseCursor(imgui.MouseCursor.Arrow)
    end

    start_y = start_y + 45
    imgui.SetCursorPos(imgui.ImVec2(imgui.GetWindowWidth() - key_sizes.backspace.x - 30, start_y))
    if imgui.Button("Back", key_sizes.backspace) then
        backspace_input()
        imgui.SetMouseCursor(imgui.MouseCursor.Arrow)
    end

    imgui.PopStyleColor(4)
end

local function draw_footer()
    local footer = "NukY Mods © 2025 | v1.7"
    imgui.SetCursorPosX((imgui.GetWindowWidth() - imgui.CalcTextSize(footer).x) * 0.5)
    imgui.TextColored(imgui.ImVec4(0.6, 0.6, 0.6, 1.0), footer)
end

local function draw_rules_categories_window()
    if not show_rules_categories[0] then return end

    imgui.SetNextWindowPos(imgui.ImVec2(700, 250), imgui.Cond.FirstUseEver)
    imgui.SetNextWindowSize(imgui.ImVec2(350, 600), imgui.Cond.FirstUseEver)

    local style = imgui.GetStyle()
    local original_rounding = style.WindowRounding
    local original_border = style.WindowBorderSize
    style.WindowRounding = 15.0
    style.WindowBorderSize = 5.0

    imgui.PushStyleColor(imgui.Col.Border, border_colors[border_color_index])
    imgui.Begin("Categorias de Regras", show_rules_categories, imgui.WindowFlags.NoCollapse)

    imgui.TextColored(imgui.ImVec4(0.0, 1.0, 1.0, 1.0), "SELECIONE UMA CATEGORIA:")
    imgui.Separator()
    imgui.Spacing()

    local categories = {
        "UCP/Páginas/Regras",
        "PROIBIÇÕES GERAIS",
        "CHAT",
        "VOIP",
        "TERRITÓRIOS",
        "BANCO",
        "SEQUESTROS",
        "ASSALTO A REFÉM (/AB)",
        "DESMANCHE",
        "CONTA",
        "ÁREAS SAFES",
        "INVASÃO DE FAVELA",
        "REGRAS ADICIONAIS DO ROLEPLAY",
        "REGRAS DE ORGANIZAÇÕES E CORPORAÇÕES",
        "CORREGEDORIA DE JUSTIÇA E TRIBUNAL FEDERAL",
        "CONFEDERAÇÃO CRIMINOSA E TRIBUNAL DO CRIME",
        "REGRAS GERAIS DE DENÚNCIAS NO SITE"
    }

    for i, category in ipairs(categories) do
        if imgui.Button(category, imgui.ImVec2(320, 30)) then
            selected_category[0] = i - 1
            show_rules_window[0] = true
        end
        imgui.Spacing()
    end

    imgui.End()
    style.WindowRounding = original_rounding
    style.WindowBorderSize = original_border
    imgui.PopStyleColor()
end

local function draw_rules_window()
    if not show_rules_window[0] then return end

    imgui.SetNextWindowPos(imgui.ImVec2(700, 250), imgui.Cond.FirstUseEver)
    imgui.SetNextWindowSize(imgui.ImVec2(650, 500), imgui.Cond.FirstUseEver)

    local style = imgui.GetStyle()
    local original_rounding = style.WindowRounding
    local original_border = style.WindowBorderSize
    style.WindowRounding = 15.0
    style.WindowBorderSize = 5.0

    imgui.PushStyleColor(imgui.Col.Border, border_colors[border_color_index])
    imgui.Begin("REGRAS DO SERVIDOR", show_rules_window, imgui.WindowFlags.NoCollapse)

    local categories = {
        "UCP/Páginas/Regras",
        "PROIBIÇÕES GERAIS",
        "CHAT",
        "VOIP",
        "TERRITÓRIOS",
        "BANCO",
        "SEQUESTROS",
        "ASSALTO A REFÉM (/AB)",
        "DESMANCHE",
        "CONTA",
        "ÁREAS SAFES",
        "INVASÃO DE FAVELA",
        "REGRAS ADICIONAIS DO ROLEPLAY",
        "REGRAS DE ORGANIZAÇÕES E CORPORAÇÕES",
        "CORREGEDORIA DE JUSTIÇA E TRIBUNAL FEDERAL",
        "CONFEDERAÇÃO CRIMINOSA E TRIBUNAL DO CRIME",
        "REGRAS GERAIS DE DENÚNCIAS NO SITE"
    }
    local selected = categories[selected_category[0] + 1]
    imgui.TextColored(imgui.ImVec4(0.0, 1.0, 1.0, 1.0), selected)

    imgui.BeginChild("RulesContent", imgui.ImVec2(0, 0), true, imgui.WindowFlags.AlwaysVerticalScrollbar)
    for _, rule in ipairs(server_rules[selected]) do
        imgui.TextWrapped(rule)
        imgui.Separator()
    end
    imgui.EndChild()

    imgui.End()
    style.WindowRounding = original_rounding
    style.WindowBorderSize = original_border
    imgui.PopStyleColor()
end

local function draw_jail_categories_window()
    if not show_jail_categories[0] then return end

    imgui.SetNextWindowPos(imgui.ImVec2(700, 250), imgui.Cond.FirstUseEver)
    imgui.SetNextWindowSize(imgui.ImVec2(350, 600), imgui.Cond.FirstUseEver)

    local style = imgui.GetStyle()
    local original_rounding = style.WindowRounding
    local original_border = style.WindowBorderSize
    style.WindowRounding = 15.0
    style.WindowBorderSize = 5.0

    imgui.PushStyleColor(imgui.Col.Border, border_colors[border_color_index])
    imgui.Begin("Categorias de Cadeias", show_jail_categories, imgui.WindowFlags.NoCollapse)

    imgui.TextColored(imgui.ImVec4(0.0, 1.0, 1.0, 1.0), "SELECIONE UMA CATEGORIA:")
    imgui.Separator()
    imgui.Spacing()

    local categories = {
        "10 MINUTOS",
        "50 MINUTOS",
        "80 MINUTOS",
        "100 MINUTOS",
        "150 MINUTOS",
        "180 MINUTOS",
        "200 MINUTOS",
        "300 MINUTOS",
        "KICKS",
        "BANIMENTOS TEMPORÁRIOS",
        "BANIMENTOS PERMANENTES",
        "Tabela da censura (Mute Temporário de comunicação)"
    }

    for i, category in ipairs(categories) do
        if imgui.Button(category, imgui.ImVec2(320, 30)) then
            selected_category[0] = i - 1
            show_jail_window[0] = true
        end
        imgui.Spacing()
    end

    imgui.End()
    style.WindowRounding = original_rounding
    style.WindowBorderSize = original_border
    imgui.PopStyleColor()
end

local function draw_jail_window()
    if not show_jail_window[0] then return end

    imgui.SetNextWindowPos(imgui.ImVec2(700, 250), imgui.Cond.FirstUseEver)
    imgui.SetNextWindowSize(imgui.ImVec2(650, 500), imgui.Cond.FirstUseEver)

    local style = imgui.GetStyle()
    local original_rounding = style.WindowRounding
    local original_border = style.WindowBorderSize
    style.WindowRounding = 15.0
    style.WindowBorderSize = 5.0

    imgui.PushStyleColor(imgui.Col.Border, border_colors[border_color_index])
    imgui.Begin("TABELA DE PUNIÇÕES", show_jail_window, imgui.WindowFlags.NoCollapse)

    local categories = {
        "10 MINUTOS",
        "50 MINUTOS",
        "80 MINUTOS",
        "100 MINUTOS",
        "150 MINUTOS",
        "180 MINUTOS",
        "200 MINUTOS",
        "300 MINUTOS",
        "KICKS",
        "BANIMENTOS TEMPORÁRIOS",
        "BANIMENTOS PERMANENTES",
        "Tabela da censura (Mute Temporário de comunicação)"
    }
    local selected = categories[selected_category[0] + 1]
    imgui.TextColored(imgui.ImVec4(0.0, 1.0, 1.0, 1.0), selected)

    imgui.BeginChild("JailContent", imgui.ImVec2(0, 0), true, imgui.WindowFlags.AlwaysVerticalScrollbar)
    for _, rule in ipairs(jail_punishments[selected]) do
        imgui.TextWrapped(rule)
        imgui.Separator()
    end
    imgui.EndChild()

    imgui.End()
    style.WindowRounding = original_rounding
    style.WindowBorderSize = original_border
    imgui.PopStyleColor()
end

local function draw_discord_categories_window()
    if not show_discord_categories[0] then return end

    imgui.SetNextWindowPos(imgui.ImVec2(700, 250), imgui.Cond.FirstUseEver)
    imgui.SetNextWindowSize(imgui.ImVec2(350, 600), imgui.Cond.FirstUseEver)

    local style = imgui.GetStyle()
    local original_rounding = style.WindowRounding
    local original_border = style.WindowBorderSize
    style.WindowRounding = 15.0
    style.WindowBorderSize = 5.0

    imgui.PushStyleColor(imgui.Col.Border, border_colors[border_color_index])
    imgui.Begin("Categorias de Regras Discord", show_discord_categories, imgui.WindowFlags.NoCollapse)

    imgui.TextColored(imgui.ImVec4(0.0, 1.0, 1.0, 1.0), "SELECIONE UMA CATEGORIA:")
    imgui.Separator()
    imgui.Spacing()

    local categories = {
        "REGRAS GERAIS DO DISCORD",
        "TICKETS"
    }

    for i, category in ipairs(categories) do
        if imgui.Button(category, imgui.ImVec2(320, 30)) then
            selected_category[0] = i - 1
            show_discord_window[0] = true
        end
        imgui.Spacing()
    end

    imgui.End()
    style.WindowRounding = original_rounding
    style.WindowBorderSize = original_border
    imgui.PopStyleColor()
end

local function draw_discord_window()
    if not show_discord_window[0] then return end

    imgui.SetNextWindowPos(imgui.ImVec2(700, 250), imgui.Cond.FirstUseEver)
    imgui.SetNextWindowSize(imgui.ImVec2(650, 500), imgui.Cond.FirstUseEver)

    local style = imgui.GetStyle()
    local original_rounding = style.WindowRounding
    local original_border = style.WindowBorderSize
    style.WindowRounding = 15.0
    style.WindowBorderSize = 5.0

    imgui.PushStyleColor(imgui.Col.Border, border_colors[border_color_index])
    imgui.Begin("REGRAS DO DISCORD", show_discord_window, imgui.WindowFlags.NoCollapse)

    local categories = {
        "REGRAS GERAIS DO DISCORD",
        "TICKETS"
    }
    local selected = categories[selected_category[0] + 1]
    imgui.TextColored(imgui.ImVec4(0.0, 1.0, 1.0, 1.0), selected)

    imgui.BeginChild("DiscordContent", imgui.ImVec2(0, 0), true, imgui.WindowFlags.AlwaysVerticalScrollbar)
    for _, rule in ipairs(discord_rules[selected]) do
        imgui.TextWrapped(rule)
        imgui.Separator()
    end
    imgui.EndChild()

    imgui.End()
    style.WindowRounding = original_rounding
    style.WindowBorderSize = original_border
    imgui.PopStyleColor()
end

local function draw_search_window()
    if not show_search_window[0] then return end

    imgui.SetNextWindowPos(imgui.ImVec2(700, 250), imgui.Cond.FirstUseEver)
    imgui.SetNextWindowSize(imgui.ImVec2(650, 500), imgui.Cond.FirstUseEver)

    local style = imgui.GetStyle()
    local original_rounding = style.WindowRounding
    local original_border = style.WindowBorderSize
    style.WindowRounding = 15.0
    style.WindowBorderSize = 5.0

    imgui.PushStyleColor(imgui.Col.Border, border_colors[border_color_index])
    imgui.Begin("Resultados da Pesquisa", show_search_window, imgui.WindowFlags.NoCollapse)

    imgui.TextColored(imgui.ImVec4(0.0, 1.0, 1.0, 1.0), "BIBLIA RP ENCONTRADA:")
    imgui.SameLine()
    imgui.SetCursorPosX(imgui.GetWindowWidth() - 200)
    if imgui.Button("CÓDIGOS RELACIONADOS", imgui.ImVec2(180, 30)) then
        show_related_codes_window[0] = true
    end

    imgui.BeginChild("SearchContent", imgui.ImVec2(0, 0), true, imgui.WindowFlags.AlwaysVerticalScrollbar)
    for _, result in ipairs(search_results) do
        imgui.TextWrapped(result.code .. ": " .. result.desc)
        imgui.Separator()
    end
    imgui.EndChild()

    imgui.End()
    style.WindowRounding = original_rounding
    style.WindowBorderSize = original_border
    imgui.PopStyleColor()
end

local function draw_related_codes_window()
    if not show_related_codes_window[0] then return end

    imgui.SetNextWindowPos(imgui.ImVec2(700, 250), imgui.Cond.FirstUseEver)
    imgui.SetNextWindowSize(imgui.ImVec2(350, 500), imgui.Cond.FirstUseEver)

    local style = imgui.GetStyle()
    local original_rounding = style.WindowRounding
    local original_border = style.WindowBorderSize
    style.WindowRounding = 15.0
    style.WindowBorderSize = 5.0

    imgui.PushStyleColor(imgui.Col.Border, border_colors[border_color_index])
    imgui.Begin("Palavras-Chave Registradas", show_related_codes_window, imgui.WindowFlags.NoCollapse)

    imgui.TextColored(imgui.ImVec4(0.0, 1.0, 1.0, 1.0), "PALAVRAS-CHAVE DISPONÍVEIS:")

    imgui.BeginChild("RelatedCodesContent", imgui.ImVec2(0, 0), true, imgui.WindowFlags.AlwaysVerticalScrollbar)
    for keyword, _ in pairs(keyword_mappings) do
        imgui.TextWrapped(keyword)
        imgui.Separator()
    end
    imgui.EndChild()

    imgui.End()
    style.WindowRounding = original_rounding
    style.WindowBorderSize = original_border
    imgui.PopStyleColor()
end

imgui.OnFrame(function() return v[0] end, function()
    imgui.SetNextWindowPos(imgui.ImVec2(400, 250), imgui.Cond.FirstUseEver)
    imgui.SetNextWindowSize(imgui.ImVec2(600, 600), imgui.Cond.FirstUseEver)

    local style = imgui.GetStyle()
    local original_rounding = style.WindowRounding
    local original_border = style.WindowBorderSize
    style.WindowRounding = 15.0
    style.WindowBorderSize = 5.0

    imgui.PushStyleColor(imgui.Col.Border, border_colors[border_color_index])
    imgui.Begin("MENU CHECAR INFO by NukY", nil, imgui.WindowFlags.NoCollapse + imgui.WindowFlags.NoResize + imgui.WindowFlags.NoTitleBar)

    update_animations(imgui.GetIO().DeltaTime)
    if border_color_index ~= imgui.GetStyle().Colors[imgui.Col.Border] then
        imgui.PopStyleColor()
        imgui.PushStyleColor(imgui.Col.Border, border_colors[border_color_index])
    end

    draw_title()
    draw_input_field()
    draw_rules_button()

    imgui.SetCursorPosY(190)
    imgui.BeginChild("Resultados", imgui.ImVec2(0, 350), true, imgui.WindowFlags.AlwaysVerticalScrollbar)
    imgui.TextColored(imgui.ImVec4(0.0, 1.0, 1.0, 1.0), "BEM-VINDO AO MENU CHECAR INFO")
    imgui.Spacing()
    draw_keyboard()
    imgui.EndChild()

    imgui.Separator()
    draw_footer()

    imgui.End()
    style.WindowRounding = original_rounding
    style.WindowBorderSize = original_border
    imgui.PopStyleColor()

    draw_rules_categories_window()
    draw_rules_window()
    draw_jail_categories_window()
    draw_jail_window()
    draw_discord_categories_window()
    draw_discord_window()
    draw_search_window()
    draw_related_codes_window()
end)

function main()
    while true do
        wait(0)
        if isSampAvailable() then
            sampRegisterChatCommand("pv", function()
                v[0] = not v[0]
            end)
        end
    end
end
