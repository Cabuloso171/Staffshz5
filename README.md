-- BY NUKY - MENU DE INFORMACOES HZ5
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
    ["IC"] = "DENTRO DO JOGO, ONDE VOCE DESENVOLVE E INTERPRETA SEU PERSONAGEM.",
    ["OOC"] = "FORA DO JOGO, O MUNDO REAL, SEPARADO DO MUNDO DO SEU PERSONAGEM.",
    ["ANTI-RP"] = "NAO SEGUIR O RP, FAZER O QUE NAO FARIA NA VIDA REAL | 200 MINUTOS DE CADEIA",
    ["PK"] = "MORTE OFICIAL DO PERSONAGEM. PERDE TODA A MEMORIA DO QUE LEVOU A MORTE.",
    ["BH"] = "ANDAR PULANDO PARA TER VANTAGEM DE VELOCIDADE OU PARA FUGIR.",
    ["CB"] = "FORCAR PERSEGUICAO POLICIAL, CHAMANDO ATENCAO PARA QUERER DAR FUGA.",
    ["CJ"] = "ROUBAR UM VEICULO TIRANDO ALGUEM DA POSICAO DE MOTORISTA SEM MOTIVO APARENTE | 200 MINUTOS DE CADEIA",
    ["CL"] = "SAIR DO SERVIDOR EM MEIO A ACAO PARA SE BENEFICIAR | 1 DIA DE BAN",
    ["DB"] = "ATIRAR PELA JANELA DE UM VEICULO EM MOVIMENTO | 200 MINUTOS DE CADEIA",
    ["DM"] = "FERIR, ATIRAR OU MATAR ALGUEM SEM MOTIVO | 200 MINUTOS DE CADEIA",
    ["RDM"] = "COMETER DM CONTRA VARIOS PLAYERS CONSECUTIVAMENTE | 200 MINUTOS DE CADEIA",
    ["FP"] = "FINALIZAR PLAYER.",
    ["AFK"] = "DEIXAR O PERSONAGEM PARADO POR MOTIVOS OOC | KICK",
    ["RT"] = "O RELOGIO DO PAY DAY TRAVA, INDICANDO UM LAG OU CRASH NO SERVIDOR | KICK",
    ["HK"] = "USAR AS HELICES DO HELICOPTERO COMO ARMA PARA ATACAR OU MATAR JOGADORES | 200 MINUTOS DE CADEIA",
    ["KOS"] = "TENTAR OU MATAR ALGUEM QUE VOCE RECONHECEU AO AVISTAR A ROUPA OU ID | 200 MINUTOS DE CADEIA",
    ["MG"] = "MISTURAR IC COM OOC | 200 MINUTOS DE CADEIA",
    ["MIX"] = "MISTURAR CHAT OOC COM CHAT IC | 200 MINUTOS DE CADEIA",
    ["NRA"] = "PUXAR ARMA, ATIRAR OU FERIR UM JOGADOR EM AREA SAFE | 60 MINUTOS DE CADEIA",
    ["PG"] = "REALIZAR ALGO QUE SERIA HUMANAMENTE IMPOSSIVEL NA VIDA REAL | 200 MINUTOS DE CADEIA",
    ["SURF"] = "FICAR EM CIMA DE UM VEICULO EM MOVIMENTO, QUEBRANDO A FISICA | 10 MINUTOS DE CADEIA",
    ["VDM"] = "USAR UM VEICULO COMO ARMA PARA FERIR OU MATAR OUTRO JOGADOR | 200 MINUTOS DE CADEIA",
    ["MUC"] = "USAR CHATS E MEIOS DE COMUNICACAO ESPECIFICOS DE FORMA INAPROPRIADA OU ERRADA | 80 MINUTOS DE CADEIA",
    ["DARKRP"] = "REALIZAR ACOES QUE ENVOLVAM ASSESIO, TORTURA, IMPORTUNACAO SEXUAL, SUICIDIO, RACISMO, LGBTQIA+FOBIA, XENOFOBIA, INTOLERANCIA RELIGIOSA, ETC | 300 MINUTOS DE CADEIA",
    ["NS"] = "JOGAR SEM TER AMOR A VIDA, ARRISCANDO A VIDA DO PERSONAGEM SEM MOTIVO LOGICO | 200 MINUTOS DE CADEIA",
    ["RK"] = "SE VINGAR DE UMA ACAO OU PESSOA QUE LEVOU A SUA MORTE ANTERIORMENTE | 200 MINUTOS DE CADEIA",
    ["ASM"] = "AGREDIR UM PLAYER SEM UM MOTIVO APARENTE | 80 MINUTOS DE CADEIA",
    ["RPFTW"] = "FAZER O RP DE SEMPRE GANHAR, NUNCA ACEITANDO PERDER.",
    ["MF"] = "CRIACAO DE CONTAS OU QUALQUER ATO DE ABUSO COM O INTUITO DE FARMAR DINHEIRO | BAN PERMANENTE"
}

local keyword_mappings = {
    ["ARMA"] = {"VDM", "NRA", "HK", "DB"},
    ["POLICIA"] = {"KOS", "CB"},
    ["VEICULO"] = {"VDM", "DB", "CJ", "SURF", "HK"},
    ["PLAYER"] = {"DM", "RDM", "ASM", "FP", "PK", "RK", "NS"},
    ["MORTE"] = {"DM", "RDM", "KOS", "HK", "PK", "RK"},
    ["ACAO"] = {"ANTI-RP", "PG", "RK", "CL", "NS", "RPFTW"},
    ["CHAT"] = {"MIX", "MUC"},
    ["MOVIMENTO"] = {"BH", "SURF"},
    ["ORGANIZACAO"] = {"MF"},
    ["PRECONCEITO"] = {"DARKRP"}
}

local server_rules = {
    ["UCP/Paginas/Regras"] = {
        "1. PROIBIDO DIVULGAR INFORMACOES PESSOAIS DE OUTROS JOGADORES",
        "2. PROIBIDO QUALQUER TIPO DE DISCRIMINACAO OU PRECONCEITO",
        "3. RESPEITAR TODOS OS MEMBROS DA COMUNIDADE"
    },
    ["PROIBICOES GERAIS"] = {
        "1. PROIBIDO COMERCIO EXTERNO, INCLUINDO A VENDA DE COINS, DINHEIRO, VEICULOS, CASAS, EMPRESAS, SKINS, ACESSORIOS OU CONTAS.",
        "2. PROIBIDO USAR NICKS OFENSIVOS OU IMPROPRIOS.",
        "3. PROIBIDO O USO DE CONTAS SECUNDARIAS PARA FARMAR BENS, ITENS OU DINHEIRO. TRANSFERENCIAS ENTRE CONTAS NAO SAO PERMITIDAS.",
        "4. PROIBIDO OFENDER JOGADORES, A ADMINISTRACAO OU O SERVIDOR.",
        "5. PROIBIDO QUALQUER TIPO DE DISCRIMINACAO, PRECONCEITO, HOMOFOBIA, XENOFOBIA OU ATOS QUE PREJUDIQUEM A IMAGEM DE ALGUEM.",
        "6. PROIBIDO DIVULGAR OUTROS SERVIDORES OU LINKS EXTERNOS.",
        "7. PROIBIDO EXPLORAR BUGS OU LAGS PARA OBTER VANTAGENS.",
        "8. PROIBIDO USAR MODIFICACOES OU TRAPACAS QUE TRAGAM BENEFICIOS INDEVIDOS.",
        "9. PROIBIDO OFENSAS, CALUNIAS, DIFAMACOES E DISCRIMINACOES OOC.",
        "10. PROIBIDO ABORDAR (/AB) EM LOCAIS SAFES, EXCETO POLICIAIS.",
        "11. PROIBIDO O USO DE SNIPER, EXCETO EM TERRITORIOS E INVASOES DE FAVELAS.",
        "12. PROIBIDO SPAWN KILL (MATAR JOGADORES AO ENTRAR/SAIR DE INTERIORES OU FAVELAS EM ABORDAGENS).",
        "13. PROIBIDO USAR TASER EM TROCAS DE TIROS.",
        "14. PROIBIDO FAZER ACAO DE PATRULHAMENTO SOLO, OBRIGATORIO A PRESENCA DE DOIS OU MAIS POLICIAIS.",
        "15. PROIBIDO O USO DE BOMBAS (C4) COM INTUITO DE MATAR OUTRO(S) PLAYER(S).",
        "16. PROIBIDO O USO DE MINA TERRESTRE EM EVENTOS.",
        "17. PROIBIDO INICIAR ACAO DE LOJA COM MENOS DE 3 JOGADORES.",
        "18. PROIBIDO INICIAR QUALQUER TIPO DE ACAO/SAQUEAR CONTRA POLICIAIS, MECANICOS E SAMUS FARDADOS.",
        "19. LIDERES E SUB-LIDERES TEM OBRIGACAO DE MANTER ATIVIDADES DENTRO DA CIDADE, CASO AMBOS FIQUEM 3 DIAS OU MAIS AUSENTES A ORG/CORP SERA RESETADA.",
        "20. PROIBIDO CORRER PARA UMA AREA SAFE/INTERIOR OU FAVELA APOS TER SIDO INICIADO UMA ACAO FORA DELA (/AB OU TROCACAO DE TIRO).",
        "21. PROIBIDO A UTILIZACAO DE JBL EM AREA SAFE",
        "22. PROIBIDO PORTAR ARMAS MEDIAS/PESADAS, COMETER CRIMES E/OU INTERVIR EM ACOES CRIMINOSAS A PAISANA",
        "23. PROIBIDO APREENDER BARRAQUINHAS EM AREAS DE DESMANCHE/FAVELA, EXCETO EM DOMINACAO DE FAVELA.",
        "24. PROIBIDO A COBRANCA DE QUALQUER VALOR/ITEM PARA EFETUAR O RECRUTAMENTO DE UM PLAYER, SEJA CORPORACAO, ORGANIZACAO, HOSPITAL E/OU MECANICA.",
        "25. PROIBIDO A UTILIZACAO DE MOTIVOS INVALIDOS PARA (/ALGEMAR), OBRIGATORIO UM MOTIVO COERENTE.",
        "26. LIDERES/SUB-LIDERES DE ORGANIZACAO/CORPORACAO QUE INFRINGIREM ALGUMA REGRA DO SERVIDOR ESTARAO GERANDO 1 (UM) AVISO PARA A ORGANIZACAO/CORPORACAO NA QUAL LIDERAM.",
        "27. MEMBROS DE ORGANIZACAO/CORPORACAO QUE FOREM BANIDOS, IRAO GERAR 1 (UMA) ADVERTENCIA PARA A ORGANIZACAO/CORPORACAO NA QUAL PARTICIPA.",
        "28. LIDERES QUE FOREM BANIDOS TEMPORARIAMENTE OU PERMANENTEMENTE RECEBERAO 3 (TRES) AVISOS, SUB-LIDERES RECEBERAO APENAS 1 AVISO.",
        "29. PROIBIDO PUXAR VEICULOS VIP DURANTE ACOES.",
        "30. PARA ACOES DE TR, O TEMPO DEVERA SER RESPEITADO, PROIBIDO ATIRAR ANTES DA CONTAGEM INICIAL TERMINAR.",
        "31. PARA ACOES DE CASA ROUBAVEL, PODERAO INICIAR TROCACAO SOMENTE NO MOMENTO QUE DER O ANUNCIO DA CASA NO CHAT GLOBAL.",
        "32. A CORDA SO PODE SER UTILIZADA EM ACOES DE SEQUESTRO OU EM ACOES DE BANCO ENVOLVENDO REFENS, EXCETO DURANTE TROCA DE TIROS. CASO O REFEM SEJA UM POLICIAL, ELE SO PODERA SER RENDIDO SE ESTIVER SOZINHO.",
        "33. PROIBIDO ALGEMA DURANTE UMA TROCACAO DE TIRO.",
        "34. PROIBIDO FECHAR ENTRADAS COM VEICULOS OU QUALQUER TIPO DE OBJETOS.",
        "35. E PROIBIDO INVADIR CASAS (MAPEADAS OU NAO). A INVASAO OU ABORDAGEM SO PODERA SER FEITA POR POLICIAIS MEDIANTE A UM MANDADO DE PRISAO."
    },
    ["CHAT"] = {
        "1. PROIBIDO FLOOD E SPAM.",
        "2. PROIBIDO USAR A OLX PARA ASSUNTOS QUE NAO ENVOLVEM VENDAS LEGAIS.",
        "3. PROIBIDO USAR COMANDOS DE ANUNCIOS PARA CONVERSAR."
    },
    ["VOIP"] = {
        "1. PROIBIDO TOCAR MUSICA NO VOIP, EXCETO EM LOCAIS RESERVADOS (CASA, EMPRESA E/OU FESTAS PARTICULARES).",
        "2. PROIBIDO USAR O VOIP ENQUANTO ESTIVER FERIDO."
    },
    ["TERRITORIOS"] = {
        "1. O MINIMO PARA INICIAR UM TERRITORIO E DE 3 JOGADORES."
    },
    ["BANCO"] = {
        "1. PROIBIDO MATAR O REFEM ANTES DA NEGOCIACAO.",
        "2. O MINIMO PARA INICIAR UM ASSALTO AO BANCO E DE 5 JOGADORES."
    },
    ["SEQUESTROS"] = {
        "1. PROIBIDO SEQUESTRAR SEM INICIAR A ACAO (/AB).",
        "2. O MINIMO PARA INICIAR UM SEQUESTRO E DE 3 JOGADORES."
    },
    ["ASSALTO A REFEM (/AB)"] = {
        "1. PROIBIDO REAGIR AO ASSALTO ANTES DE 10 SEGUNDOS DA ABORDAGEM.",
        "2. PROIBIDO EXIGIR DINHEIRO DO BANCO OU ITENS QUE A VITIMA NAO POSSUA NO MOMENTO."
    },
    ["DESMANCHE"] = {
        "1. PROIBIDO ABORDAR (/AB) EM DESMANCHES, EXCETO QUANDO O LOCAL ESTIVER SOB DOMINACAO DE TERRITORIO, ACOES DE CAIXA OU CASA ROUBAVEL."
    },
    ["CONTA"] = {
        "1. A CONTA E PESSOAL E INTRANSFERIVEL. CASO SEJA PUNIDA OU BANIDA, O SERVIDOR NAO SE RESPONSABILIZA POR SEU USO."
    },
    ["AREAS SAFES"] = {
        "1. AREAS SEGURAS INCLUEM: HOSPITAIS, OFICINAS MECANICAS, HQS DE PROFISSOES, PRACAS, FABRICAS DE MATERIAIS/DROGAS, HOTEIS (APENAS INTERIOR), CONCESSIONARIAS, DELEGACIAS, DESMANCHES E BANCOS (EXCETO DURANTE ASSALTOS).",
        "2. INTERIORES DE CASAS E EMPRESAS NAO SAO AREAS SEGURAS."
    },
    ["INVASAO DE FAVELA"] = {
        "1. O MINIMO PARA INICIAR UMA INVASAO DE FAVELA E DE 5 JOGADORES.",
        "2. A INVASAO DEVE SER MARCADA COM 15 MINUTOS DE ANTECEDENCIA E REGISTRADA NO DISCORD COM UMA PRINT E UM DOS MOTIVOS VALIDOS, COMO ANUNCIOS DE VENDAS NA FAVELA ALVO PELO CHAT GLOBAL."
    },
    ["BLITZ"] = {
        "1. O MINIMO DE POLICIAIS PARA INICIAR UMA BLITZ, SAO DE 4 (QUATRO) JOGADORES; OS MESMOS DEVERAO PERMANECER ATE O FINAL DA MESMA.",
        "2. APENAS MEMBROS DE CORPORACAO PODERAO VOLTAR A BLITZ CASO MORRAM NO LOCAL DA BLITZ."
    }
}

local jail_punishments = {
    ["60 MINUTOS"] = {
        "- NRA (ATIRAR SEM ANUNCIAR)",
        "- JBL HP (APOS 1 KICK)",
        "- JBL EM AEROPORTO (APOS 1 KICK)"
    },
    ["80 MINUTOS"] = {
        "- FLOOD (APOS 3 VEZES)",
        "- ASM (AGREDIR SEM MOTIVO)",
        "- MUCS (ATENDIMENTO, DUVIDA, MISSA, NEWS, OLX, /REPORTAR)",
        "- /AN CONVERSAS/FARPAs"
    },
    ["150 MINUTOS"] = {
        "- LOJA SOLO/EM DOIS"
    },
    ["200 MINUTOS"] = {
        "- DB",
        "- DM (FERIR / MATAR SEM MOTIVO)",
        "- ACAO EM SAFE",
        "- ACAO DESMANCHE FORA DE TR OU ALGEMANDO EM ACAO",
        "- AB DESMANCHE",
        "- KOS",
        "- MG",
        "- PG (EM ACAO)",
        "- TK",
        "- VDM (ATROPELAR DE PROPOSITO)",
        "- HK",
        "- PTR SOLO",
        "- SNIPER EM ACAO DE RUA",
        "- ASSALTO A BANCO COM MENOS DE 5 PESSOAS",
        "- ANTI-RP",
        "- INVASAO SEM MARCAR",
        "- NS (NO SENSE)",
        "- RDM (RANDOM DEATHMATCH)",
        "- RK (REVENGE KILL)",
        "- SPAM KILL",
        "- CORRER PARA INTERIOR EM ACAO",
        "- FUGA PARA INTERIOR APOS ACAO (SAFE/FAVELA/INTERIOR)"
    },
    ["300 MINUTOS"] = {
        "- CORRUPCAO (SEM CONTEXTO RP)",
        "- DARK RP"
    },
    ["KICKS"] = {
        "- RT / BUGADO (SE FOR SOLICITADO PELO JOGADOR)",
        "- BUGANDO / ATRAPALHANDO EVENTO"
    },
    ["BANIMENTOS TEMPORARIOS"] = {
        "- MA CONDUTA (STAFF): 16 DIAS",
        "- CORTANDO ANIMACAO: 15 DIAS",
        "- HANDLING: 5 DIAS",
        "- ANIMACAO VANTAJOSA: 5 DIAS",
        "- ANTI-RP EXTREMO: 15 DIAS",
        "- COMBAT LOG (CL): 1 DIA"
    },
    ["BANIMENTOS PERMANENTES"] = {
        "- CHEATER (MODS PROIBIDOS)",
        "- ABUSO DE BUGS",
        "- COMERCIO ILEGAL",
        "- DIVULGACAO DE SERVIDORES",
        "- OFENSA AO SERVIDOR",
        "- NICK IMPROPRIO",
        "- MONEY FARM"
    },
    ["TABELA DE CENSURA (MUTE TEMPORARIO)"] = {
        "- FARPA FRACA (FRACO, CORNO, GADO...): PERMITIDO",
        "- FARPA MEDIA (ARROMBADO, LIXO...): 1 DIA",
        "- BULLYING (APELIDOS/COMPARACOES): 1 DIA",
        "- FLOOD APOS PRISAO: 1 DIA",
        "- BAIXO CALAO (VIADO, FDP, ETC): 2 DIAS",
        "- PRECONCEITO COM DEFICIENCIA: 3 DIAS",
        "- CONTEUDO SEXUAL/EXPLICITO: 3 DIAS",
        "- PRECONCEITO COM DOENCAS: 5 DIAS",
        "QUALQUER VOCABULARIO MAIS PESADO, COM INTUITO DE ASSEDIAR OU CONTER PRECONCEITO SERA MOTIVO DE BANIMENTO + CALAR DE 30 DIAS"
    }
}

local discord_rules = {
    ["REGRAS GERAIS DO DISCORD"] = {
        "1. PROIBIDO QUALQUER TIPO DE OFENSA OU DESRESPEITO AOS MEMBROS E STAFFS.",
        "2. PROIBIDO A DIVULGACAO DE SERVIDORES, PRODUTOS OU CONTEUDOS EXTERNOS.",
        "3. PROIBIDO O USO DE NICKS OFENSIVOS.",
        "4. PROIBIDO FLOOD, SPAM E USO DE BOTS SEM PERMISSAO.",
        "5. PROIBIDO QUALQUER CONTEUDO SEXUAL, RACISTA, HOMOFOBICO OU QUE INCITE A VIOLENCIA."
    },
    ["TICKETS"] = {
        "1. APOS CRIAR UM TICKET, AGUARDE O ATENDIMENTO. PROIBIDO MARCAR STAFFS.",
        "2. PROIBIDO CRIAR TICKETS SEM MOTIVO OU COM INTUITO DE ATRAPALHAR."
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
        local found_in_keywords = false
        if keyword_mappings[last_search_keyword] then
            found_in_keywords = true
            for _, code in ipairs(keyword_mappings[last_search_keyword]) do
                if bible_rp[code] then
                    table.insert(search_results, {code = code, desc = bible_rp[code]})
                end
            end
        else
            for code, desc in pairs(bible_rp) do
                if desc:lower():find(last_search_keyword) then
                    table.insert(search_results, {code = code, desc = desc})
                end
            end
        end

        if #search_results > 0 then
            show_search_window[0] = true
        else
            sampAddChatMessage("NENHUM RESULTADO ENCONTRADO PARA '" .. text .. "'!", 0xFFFF)
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
    if imgui.Button("ENTER", key_sizes.enter) then
        process_input()
        imgui.SetMouseCursor(imgui.MouseCursor.Arrow)
    end

    start_y = start_y + 50
    imgui.SetCursorPos(imgui.ImVec2((imgui.GetWindowWidth() - key_sizes.space.x) * 0.5, start_y))
    if imgui.Button("ESPACO", key_sizes.space) then
        append_to_input(" ")
        imgui.SetMouseCursor(imgui.MouseCursor.Arrow)
    end

    start_y = start_y + 45
    imgui.SetCursorPos(imgui.ImVec2(imgui.GetWindowWidth() - key_sizes.backspace.x - 30, start_y))
    if imgui.Button("BACK", key_sizes.backspace) then
        backspace_input()
        imgui.SetMouseCursor(imgui.MouseCursor.Arrow)
    end

    imgui.PopStyleColor(4)
end

local function draw_footer()
    local footer = "NUKY MODS © 2025 | V1.7"
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
    imgui.Begin("CATEGORIAS DE REGRAS", show_rules_categories, imgui.WindowFlags.NoCollapse)

    imgui.TextColored(imgui.ImVec4(0.0, 1.0, 1.0, 1.0), "SELECIONE UMA CATEGORIA:")
    imgui.Separator()
    imgui.Spacing()

    local categories = {
        "UCP/PAGINAS/REGRAS",
        "PROIBICOES GERAIS",
        "CHAT",
        "VOIP",
        "TERRITORIOS",
        "BANCO",
        "SEQUESTROS",
        "ASSALTO A REFEM (/AB)",
        "DESMANCHE",
        "CONTA",
        "AREAS SAFES",
        "INVASAO DE FAVELA",
        "BLITZ"
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
        "UCP/PAGINAS/REGRAS",
        "PROIBICOES GERAIS",
        "TOCPICO 02 - CHAT",
        "TOPICO 03 - VOIP",
        "TOPICO 04 - TERRITORIOS",
        "TOPICO 05 - BANCO",
        "TOPICO 06 - SEQUESTROS",
        "TOPICO 07 - ASSALTO A REFEM (/AB)",
        "TOPICO 08 - DESMANCHE",
        "TOPICO 09 - CONTA",
        "TOPICO 10 - AREAS SAFES",
        "TOPICO 11 - INVASAO DE FAVELA",
        "TOPICO 12 - BLITZ"
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
    imgui.Begin("CATEGORIAS DE CADEIAS", show_jail_categories, imgui.WindowFlags.NoCollapse)

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
        "BANIMENTOS TEMPORARIOS",
        "BANIMENTOS PERMANENTES",
        "TABELA DE CENSURA (MUTE TEMPORARIO)"
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
    imgui.Begin("TABELA DE PUNICOES", show_jail_window, imgui.WindowFlags.NoCollapse)

    local categories = {
        "60 MINUTOS",
        "80 MINUTOS",
        "150 MINUTOS",
        "200 MINUTOS",
        "300 MINUTOS",
        "KICKS",
        "BANIMENTOS TEMPORARIOS",
        "BANIMENTOS PERMANENTES",
        "TABELA DE CENSURA (MUTE TEMPORARIO)"
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
    imgui.Begin("CATEGORIAS DE REGRAS DISCORD", show_discord_categories, imgui.WindowFlags.NoCollapse)

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
    imgui.Begin("RESULTADOS DA PESQUISA", show_search_window, imgui.WindowFlags.NoCollapse)

    imgui.TextColored(imgui.ImVec4(0.0, 1.0, 1.0, 1.0), "BIBLIA RP ENCONTRADA:")
    imgui.SameLine()
    imgui.SetCursorPosX(imgui.GetWindowWidth() - 200)
    if imgui.Button("CODIGOS RELACIONADOS", imgui.ImVec2(180, 30)) then
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
    imgui.Begin("PALAVRAS-CHAVE REGISTRADAS", show_related_codes_window, imgui.WindowFlags.NoCollapse)

    imgui.TextColored(imgui.ImVec4(0.0, 1.0, 1.0, 1.0), "PALAVRAS-CHAVE DISPONIVEIS:")

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
