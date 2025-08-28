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
    ["IC"] = "Dentro do game. O mundo IC é onde você desenvolve e interpreta seu personagem.",
    ["OOC"] = "Fora do game. O seu personagem vive em um mundo separado do nosso. O mundo real é chamado de OOC.",
    ["ANTI-RP"] = "Não seguir o RP, desobedecer a biblia RP e fazer oque não faria em vida real. Roleplay significa interpretar um papel de personagem, onde você deverá gerenciar a sua vida com responsabilidade, consciência e bom senso, oque fugir disso é procedência maligna.",
    ["PK / PD"] = "É a morte oficial do personagem. Quando você morre, perde TODA a memória do seu personagem, não lembrando quem te matou ou qualquer evento que levou a sua morte.",
    ["BH"] = "É quando você anda pulando no jogo para ter alguma vantagem (ser mais rápido ou fugir).",
    ["CB"] = "É quando você força perseguição policial, chamando atenção para querer dar fuga.",
    ["CJ"] = "É quando você rouba um veículo tirando alguém da posição de motorista. Isso sem motivo aparente.",
    ["CL"] = "É quando você sai do servidor em meio a ação para se beneficiar. Exemplo: Sair para não ser preso, abordado e assaltado.",
    ["DB"] = "Muito semelhante com o conceito de VDM. O motorista ou passageiro não pode atirar pela janela do veículo.",
    ["DM"] = "Quando você fere, atira, ou mata alguém sem motivo.",
    ["RDM"] = "Seguindo o conceito de DM, o player não poderá cometer DM contra vários players consecutivamente.",
    ["FP"] = "Finalizar Player.",
    ["AFK"] = "Alguém que deixa seu personagem no IC parado, Por motivos OOC.",
    ["RT"] = "Quando um lag acontece ou o dá crash no servidor, para as pessoas você está parado, porém para você tudo está tranquilo e as pessoas que estão paradas. É possível perceber se está de RT com a contagem do Pay Day no canto inferior esquerdo da tela do jogo. O relógio literalmente Trava.",
    ["HK"] = "O termo HK refere-se a utilizar as hélices do helicóptero como uma arma letal para atacar/matar jogadores.",
    ["KOS"] = "É quando você tenta ou consegue matar alguém que você reconheceu ao avistar sua roupa ou ID. (Atirar em um policial só por ser policial). (Atirar em um membro de org só por ele ser de org) etc.",
    ["MG"] = "Misturar IC com OOC.",
    ["MIX"] = "O termo MIX é um termo que se encaixa no MG. É quando você mistura chat do OOC com chat IC. Exemplo: 'MINHA MÃE TÁ ME CHAMANDO /OOC / VOCÊ NÃO SABE JOGAR /OOC'.",
    ["NRA"] = "Puxar arma e/ou atirar e/ou ferir um jogador em área safe (ações não podem acontecer nesses locais).",
    ["PG"] = "É quando você realiza algo que seria humanamente impossível de se realizar na vida real.",
    ["SURF"] = "O termo SURF refere-se a praticar o ato de ficar em cima de um veículo em movimento, quebrando completamente a física.",
    ["VDM"] = "É quando um jogador usa um veículo como arma para ferir ou matar outro jogador (atropelamento proposital).",
    ["MUC"] = "Usar chats e meios de comunicação específicos de forma inapropriada ou errada. Ex: usar /anorg para farpar alguém.",
    ["DARKRP"] = "O termo Dark RP significa realizar ações que envolvam ou remetam a assédio, tortura, importunação sexual, suicídio, racismo, LGBTQIA+fobia, xenofobia, intolerância religiosa, entre outras.",
    ["NS"] = "Você precisa sempre jogar dando amor a sua vida, não arriscando a vida do personagem.",
    ["RK"] = "Conceito que entra junto com PK. É extremamente proibido você se vingar de uma ação e pessoa que levou sua morte anteriormente.",
    ["ASM"] = "O termo ASM (Agredir Sem Motivo) refere-se a praticar o ato de agressão em um player sem um motivo aparente.",
    ["RPFTW"] = "Player faz a todo o custo o RP de sempre ganhar! nunca aceita perder, o mesmo quer sempre estar à frente dos outros jogadores.",
    ["MF"] = "Criação de contas ou qualquer ato de abuso com intuito de farmar grana."
}

local keyword_mappings = {
    ["arma"] = {"VDM", "NRA", "HK", "DB"},
    ["policia"] = {"KOS", "CB"},
    ["veiculo"] = {"VDM", "DB", "CJ", "WH", "SURF", "CMP"},
    ["player"] = {"DM", "RDM", "ASM", "FP", "PK", "CK"},
    ["morte"] = {"DM", "RDM", "KOS", "HK", "PK", "RK"},
    ["acao"] = {"ANTI-RP", "PG", "RK", "NS", "CB", "CL"},
    ["chat"] = {"MUC", "MIX"},
    ["movimento"] = {"BH", "SURF"},
    ["organizacao"] = {"MF"},
    ["preconceito"] = {"DARKRP"},
    ["conta"] = {"MF"},
    ["vida"] = {"NS"}
}

local server_rules = {
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
        "14. Proibido fazer ação de patrulhamento solo, obrigatório a presença de dois ou mais policiais.",
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
        "31. Para ações de Casa roubável, poderão iniciar trocação somente no momento que der o anúncio da casa no chat global.",
        "32. A corda só pode ser utilizada em ações de sequestro ou em ações de banco envolvendo reféns, exceto durante troca de tiros. Caso o refém seja um policial, ele só poderá ser rendido se estiver sozinho.",
        "33. Proibido Algemar durante uma trocação de tiro.",
        "34. Proibido fechar entradas com veículos ou qualquer tipo de objetos.",
        "35. É proibido invadir casas (mapeadas ou não). A invasão ou abordagem só poderá ser feita por policiais mediante a um mandado de prisão."
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
        "1. O mínimo para iniciar um território é de 3 jogadores."
    },
    ["BANCO"] = {
        "1. Proibido matar o refém antes da negociação.",
        "2. O mínimo para iniciar um assalto ao banco é de 5 jogadores."
    },
    ["SEQUESTROS"] = {
        "1. Proibido sequestrar sem iniciar a ação (/ab).",
        "2. O mínimo para iniciar um sequestro é de 3 jogadores."
    },
    ["ASSALTO A REFÉM (/AB)"] = {
        "1. Proibido reagir ao assalto antes de 10 segundos da abordagem.",
        "2. Proibido exigir dinheiro do banco ou itens que a vítima não possua no momento."
    },
    ["DESMANCHE"] = {
        "1. Proibido abordar (/ab) em desmanches, exceto quando o local estiver sob dominação de território, ações de caixa ou casa roubável."
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
    ["BLITZ"] = {
        "1. O mínimo de policiais para iniciar uma blitz são de 4 (quatro) jogadores. Os mesmos deverão permanecer até o final da mesma.",
        "2. Apenas membros de corporação poderão voltar à blitz caso morram no local da blitz."
    },
    ["REGRAS GERAIS DE DENÚNCIAS NO SITE"] = {
        "1. Todas as denúncias devem estar em suas respectivas categorias. Caso não esteja, a denúncia poderá ser recusada sem análise prévia.",
        "2. A denúncia deve ser feita pelo próprio player ou por alguém que participou daquela ação em específico.",
        "3. Caso seja punido por denúncia site, terá um prazo de 12 horas para contestar a punição. Após esse prazo, não terá direito a reclamações ou revisão.",
        "4. Caso seja banido por denúncia site, terá um prazo de 24 horas para contestar a punição. Após esse prazo, não terá direito a reclamações ou revisão.",
        "5. É completamente proibido desrespeitar ou ofender um player durante uma denúncia.",
        "6. Ao realizar a denúncia, tenha ciência de que está denunciando a pessoa correta e que tem provas suficientes, a qual sua falta poderá acarretar em punição por falsa denúncia.",
        "7. Em caso de provas com vídeo, verifique se o mesmo foi enviado. Caso conste quebra de link, poderá abrir nova denúncia corrigindo ou abrir um ticket para enviar o link a um administrador.",
        "8. Após a denúncia ter sido averiguada e dadas as contra-provas, a equipe responsável avaliará o caso com base nas evidências apresentadas. Provas adicionais após a denúncia ter sido aceita ou recusada serão automaticamente rejeitadas.",
        "9. Caso o denunciante cometa algum delito em sua prova, poderá ser punido.",
        "10. Para que uma denúncia seja válida, deverá conter:",
        "   I. Provas (Vídeo).",
        "   II. RG do acusado, data, hora, logo e chat do servidor nas provas.",
        "   III. Motivo e uma breve descrição do ocorrido.",
        "   IV. Provas dentro de 24 horas após o ocorrido.",
        "   V. Não poderá ter provas editadas, cortadas e/ou com algum mod que atrapalhe a visualização.",
        "   VI. Não poderá ter provas tiradas com câmeras (celular, tablet ou outros) / prints / gravações por cima de outras gravações.",
        "   VII. Não poderá criar uma denúncia utilizando provas coletadas por outros jogadores.",
        "   VIII. Qualquer tipo de denúncia deverá conter início, meio e fim. Sendo assim, ao iniciar a ação/ocorrido, deverá começar a gravar.",
        "11. Membros de organização ou corporação que tiverem uma denúncia aceita, irão gerar uma advertência para a organização ou corporação.",
        "12. Líderes/Sub-Líderes de corporações ou organizações que tiverem uma denúncia aceita, irão gerar um aviso para a organização ou corporação.",
        "13. Denúncias em categoria de CHEATER devem conter vídeo sem edições e com o áudio original do jogo, e a gravação do TAB mostrando o ping e o ID do acusado.",
        "14. Denúncias envolvendo CL (Combat Logging) devem conter o vídeo do exato momento do quit, com 30 segundos adicionais que comprovem que foi um CL.",
        "15. Denúncias que contiverem links que não sejam do YouTube ou links de sites terceiros serão recusadas instantaneamente."
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
    ["UCP/Páginas/Regras"] = {
        "1. Proibido divulgar informações pessoais de outros jogadores",
        "2. Proibido qualquer tipo de discriminação ou preconceito",
        "3. Respeitar todos os membros da comunidade"
    }
}

local jail_punishments = {
    ["60 MINUTOS"] = {
        "- NRA (Atirar sem anunciar)",
        "- JBL HP (após 1 kick)",
        "- JBL em Aeroporto (após 1 kick)"
    },
    ["80 MINUTOS"] = {
        "- Flood (Apos 3 vezes)",
        "- ASM (Agredir sem motivo)",
        "- MUC Atendimento",
        "- MUC Duvida",
        "- MUC Missa",
        "- MUC NEWS",
        "- MUC OLX",
        "- MUC /Reportar",
        "- MUC /AN Conversas/Farpas"
    },
    ["150 MINUTOS"] = {
        "- Loja Solo/em Dois"
    },
    ["200 MINUTOS"] = {
        "- DB",
        "- DM (Ferir / Matar sem motivo)",
        "- VDM (Atropelar de propósito)",
        "- KOS",
        "- HK",
        "- PTR Solo",
        "- Sniper em Acao de Rua",
        "- Acao Safe",
        "- Acao Desmanche (fora de TR ou Algemando em Acao)",
        "- AB Desmanche",
        "- Anti-RP",
        "- MG",
        "- MIX",
        "- PG (Apenas em Acao)",
        "- TK",
        "- Invasao sem marcar",
        "- NS (SEM AMOR A VIDA)",
        "- RDM (DMS CONSECUTIVOS)",
        "- RK",
        "- Spam Kill",
        "- Fuga para interior em acao",
        "- Assalto a Banco com menos de 5 pessoas"
    },
    ["300 MINUTOS"] = {
        "- Corrupcao (se n fizer parte do RP)",
        "- Dark RP"
    },
    ["KICKS"] = {
        "- RT / BUGADO ( se for solicitado pelo jogador)",
        "- Bugando / Atrapalhando Evento",
        "- AFK (Apenas se estiver atrapalhando RP e/ou no meio da pista)",
        "- TROCA NICK (se voltar, o admin banira)",
        "- SKIN DO CJ (se voltar, admin banira o IP)"
    },
    ["BANIMENTOS TEMPORÁRIOS"] = {
        "1 DIA",
        "- CL",
        "",
        "5 dias",
        "- Handling",
        "- Animacao vantajosa",
        "",
        "15 dias",
        "- Cortando animacao",
        "- Anti-RP extremo",
        "",
        "16 dias",
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
        "",
        "2 DIAS",
        "- Palavreados de Baixo calao como ofensa: (EX: fdp, fds, tmnc; etc...)",
        "",
        "3 DIAS",
        "- Palavreados preconceituosos com deficiencia: (EX: tem down, Teleton, tijolinho)",
        "- Utilizacao de partes sexuais, partes intimas, e/ou palavriados vulgar obscenos (mesmo se for giria 'acalma a ppk' por ex)",
        "",
        "5 Dias",
        "- Palavreados Preconceituosos, ofensas, ou deboche com Doencas Terminais.",
        "",
        "Qualquer vocabulario mais pesado, com intuito de assediar ou conter preconceito sera motivo de banimento + calar de 30 dias"
    }
}

local discord_rules = {
    ["REGRAS GERAIS DO DISCORD"] = {
        "1. Não abuse do CAPS-LOCK. O uso excessivo de letras maiúsculas é considerado gritar ou chamar atenção desnecessariamente.",
        "2. É proibido usar o chat de texto para áudio. Evite mandar uma mensagem por vez, isso pode ser considerado flood.",
        "3. Não envie mensagens com spam, isso pode sobrecarregar o chat e atrapalhar a comunicação de outros jogadores.",
        "4. É proibido divulgar qualquer tipo de conteúdo NSFW (conteúdo sexualmente explícito, nudez, pornografia ou qualquer tipo de conteúdo adulto), gore, links maliciosos ou vírus.",
        "5. Qualquer tipo de ofensa ou ataque pessoal a outro membro é proibido e pode gerar punições.",
        "6. É proibido o uso de nicks impróprios ou que causem qualquer tipo de ofensa ou constrangimento.",
        "7. É proibido fazer apologia a qualquer tipo de crime (tráfico de drogas, porte ilegal de armas, entre outros), racismo, machismo, xenofobia, homofobia ou qualquer outro tipo de preconceito.",
        "8. Se algum jogador estiver violando as regras, utilize os canais de denúncias para reportar o ocorrido. Evite fazer justiça com as próprias mãos.",
        "9. Proibido tocar música nos canais de voz."
    },
    ["TICKETS"] = {
        "1. O ticket é um canal de suporte e deve ser usado para denúncias, dúvidas ou reportar bugs.",
        "2. Evite o uso de tickets desnecessariamente. Antes de abrir, verifique se sua dúvida já foi sanada por outro membro da equipe ou jogador.",
        "3. Não ofenda a equipe no ticket, utilize uma linguagem respeitosa e clara para que sua demanda seja atendida da melhor forma possível.",
        "4. Se o ticket for aberto para denúncia, é obrigatório enviar a prova do ocorrido (vídeo ou print)."
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
        "BLITZ",
        "REGRAS GERAIS DE DENÚNCIAS NO SITE",
        "REGRAS ADICIONAIS DO ROLEPLAY",
        "REGRAS DE ORGANIZAÇÕES E CORPORAÇÕES",
        "CORREGEDORIA DE JUSTIÇA E TRIBUNAL FEDERAL",
        "CONFEDERAÇÃO CRIMINOSA E TRIBUNAL DO CRIME",
        "UCP/Páginas/Regras"
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
        "BLITZ",
        "REGRAS GERAIS DE DENÚNCIAS NO SITE",
        "REGRAS ADICIONAIS DO ROLEPLAY",
        "REGRAS DE ORGANIZAÇÕES E CORPORAÇÕES",
        "CORREGEDORIA DE JUSTIÇA E TRIBUNAL FEDERAL",
        "CONFEDERAÇÃO CRIMINOSA E TRIBUNAL DO CRIME",
        "UCP/Páginas/Regras"
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
        "60 MINUTOS",
        "80 MINUTOS",
        "150 MINUTOS",
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
        "60 MINUTOS",
        "80 MINUTOS",
        "150 MINUTOS",
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
