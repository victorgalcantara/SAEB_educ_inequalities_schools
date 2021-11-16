# SAEB_educ_inequalities_schools
Analysis of socioeconomic status and performance on national evaluation in elementary education. Some intuitive graphs and descriptive analysis about these variables.

---
title: "Boxplot_INSExDesemp"
author: Victor G. Alcantara
output: html_document
---

```{r message=F,include=F}
# 0. Packages and Setup -------------------------------------------------------

library(tidyverse)
library(ggplot2)
library(cowplot)  # for bind graphs together

# 1. Import data --------------------------------------------------------------

wd <- "C:/Users/VictorGabriel/Documents/DADOS/EDUCACAO/"

ano = seq(from=2013,to=2019,by=2) # anos em que Saeb passou a ser censitário

saeb <- list(s13 = data.frame(), s15 = data.frame(), s17 = data.frame(),
             s19 = data.frame())

for(i in 1:length(ano)) {
  saeb[[i]] <- read_csv(paste0(wd,"SAEB/",ano[i],"/DADOS/TS_ESCOLA.csv")) # abra todas as bases
  gc()
}

# 2. Data management -------------------------------------------------------

# 2.1 Select and filter ----------------------------------------------------

# SAEB
my_saeb <- list(s13 = data.frame(), s15 = data.frame(), s17 = data.frame(),
                s19 = data.frame())

for(i in 1:length(ano)) {
  my_saeb[[i]] <- saeb[[i]] %>% select(ID_ESCOLA, MEDIA_9EF_LP, MEDIA_9EF_MT, 
                                       ID_DEPENDENCIA_ADM, NIVEL_SOCIO_ECONOMICO) 
  }


# 2.2 Missing data -----------------------------------------------------------
#

for(i in 1:length(ano)) {
  my_saeb[[i]] <- na.exclude(my_saeb[[i]])
}

# 2.3 Cut and mutate ---------------------------------------------------------

# Categorias: niveis de interpretacao pedagogica utilizado no SARESP 
# e pelo Mov. Todos Pela Educação para tornar mais compreensível a
# escala SAEB

for(i in 1:length(ano)) {
my_saeb[[i]]$escala_saeb_LP        <- cut(my_saeb[[i]]$MEDIA_9EF_LP,
                                          breaks = c(0, 200, 275, 325,400),
                                          labels = c("Abaixo do básico",
                                                     "Básico",
                                                     "Adequado",
                                                     "Avançado"),
                                          ordered_result = TRUE, 
                                          right = FALSE)

}

for(i in 1:length(ano)) {
  my_saeb[[i]]$escala_saeb_MT        <- cut(my_saeb[[i]]$MEDIA_9EF_MT,
                                         breaks = c(0, 225, 300, 350,400),
                                         labels = c("Abaixo do básico",
                                                    "Básico",
                                                    "Adequado",
                                                    "Avançado"),
                                         ordered_result = TRUE, # Try without this #
                                         right = FALSE)
  
}

# Indicador de Nível Socioeconômico (INSE) -------------------------------------
# Trata-se de um indicador operacionalizado por Maria T. Alves e José F. Soares,
# ambos professores da UFMG com contribuições substantivas para a área e para
# o INEP.

for(i in 1:length(ano)) {
  my_saeb[[i]] <- rename(my_saeb[[i]], 
                         INSE = NIVEL_SOCIO_ECONOMICO)
  my_saeb[[i]]$INSE <- iconv(my_saeb[[i]]$INSE, to = 'Latin1')
}

# Transformando categorias para Baixo, Medio e Alto

# 2013
          
my_saeb[[1]] <- my_saeb[[1]] %>%  
  mutate(INSE = case_when(INSE == "Grupo 1" ~ 1,
                          INSE == "Grupo 2" ~ 1, 
                          INSE == "Grupo 3" ~ 2,
                          INSE == "Grupo 4" ~ 2,
                          INSE == "Grupo 5" ~ 2, 
                          INSE == "Grupo 6" ~ 3,
                          INSE == "Grupo 7" ~ 3
  ) )

# 2015

for(i in c(2)) {
  my_saeb[[i]] <- my_saeb[[i]] %>%  
    mutate(INSE = case_when(INSE == "Muito Baixo" ~ 1,
                            INSE == "Baixo"       ~ 1,
                            INSE == "Médio Baixo" ~ 2,
                            INSE == "Médio"       ~ 2,
                            INSE == "Médio Alto"  ~ 2,
                            INSE == "Alto"        ~ 3, 
                            INSE == "Muito Alto"  ~ 3
    ) )
}

# 2017 :  Por algum motivo não está padronizado em 7 categorias 
#         como nos outros anos. Estou forçando a barra
#         recodificando as categorias para baixo, médio e alto.
#         É necessário conferir a construção dessas categorias.
#         Talvez reconstruí-las seguindo a metodologia.

my_saeb[[3]] <- my_saeb[[3]] %>%  
  mutate(INSE = case_when(INSE == "Grupo 1" ~ 1,
                          INSE == "Grupo 2" ~ 1, 
                          INSE == "Grupo 3" ~ 2,
                          INSE == "Grupo 4" ~ 2,
                          INSE == "Grupo 5" ~ 3, 
                          INSE == "Grupo 6" ~ 3
  ) )

# 2019
for(i in c(4)) {
  my_saeb[[i]] <- my_saeb[[i]] %>%  
  mutate(INSE = case_when(INSE == "Nível I"   ~ 1,
                          INSE == "Nível II"  ~ 1,
                          INSE == "Nível III" ~ 2,
                          INSE == "Nível IV"  ~ 2,
                          INSE == "Nível V"   ~ 2,
                          INSE == "Nível VI"  ~ 3, 
                          INSE == "Nível VII" ~ 3
  ) )
}

# Ordenando as categorias
for(i in 1:length(ano)) {
my_saeb[[i]]$INSE <- factor(my_saeb[[i]]$INSE,
                     labels = c("Baixo", "Medio", "Alto"),
                     levels = c(1,2,3),
                     ordered = TRUE)
}

# Union ----------------------------------------------------------------------

for(i in 1:length(ano)) {
my_saeb[[i]] <- my_saeb[[i]] %>% mutate(ano = ano[i]) # ID para o ano da avaliação
}

# Unindo todas as bases 2013-2019 em um único banco para representar a variação temporal
mydata <- union(my_saeb[[1]],my_saeb[[2]])
mydata <- union(mydata,my_saeb[[3]])
mydata <- union(mydata,my_saeb[[4]])

```

## Análise da distribuição de desempenho médio condicionada ao Nível Socioeconômico médio das Escolas

Esta é uma análise bivariada muito informativa. Com ela podemos começar a compreender a relação entre origem social e desempenho escolar; em outras palavras, a dimensão socioeconômica das desigualdades educacionais.

Temos duas variáveis: i) proficiência/desempenho médio por Escola; ii) Indicador de nível socioeconômico (INSE)

# Proficiência/desempenho médio por Escola

A proficiência ou desempenho médio por Escola é um indicador importante do aprendizado dos alunos atendidos pela instituição. Trata-se de uma variável métrica contínua que varia na escala SAEB de 0-400. Para uma interpretação pedagógica da distribuição desta variável, consideramos os níveis de desempenho utilizadas na avaliação do Estado de São Paulo (SARESP), discutida por Soares (2018): abaixo do básico, básico, adequado e avançado. Para cada segmento da educação básica há níveis de interpretação pedagógica específicos, tal como nos quadros abaixo.

![Caption:](C:/Users/VictorGabriel/Dropbox/mestrado PPGSA_VictorAlcantara/02 scripts/análise_descritiva/boxplot_INSExDesemp/niveis_LP)
![Caption:](C:/Users/VictorGabriel/Dropbox/mestrado PPGSA_VictorAlcantara/02 scripts/análise_descritiva/boxplot_INSExDesemp/niveis_MT)

```{r message=F,include=F}
# Português ----
# Camada 1 : Histograma
histLP <- ggplot(mydata, aes(x = MEDIA_9EF_LP) )+
  theme_bw()+
  geom_histogram(bins = 30)+
  facet_grid( rows = vars(ano))

# Camada 2 : labels
histLP <- histLP +
  #scale_color_manual(values = c("brown4","turquoise1","green"))+
    scale_x_continuous(
    name = "Desempenho médio em Língua Portuguesa",
    limits = c(100,400)) +
  scale_y_continuous(
    name = "Frequência" )


# Camada 3 : linhas horizontais para os níveis de proficiência na escala SAEB
histLP <- histLP +
  # Abaixo do básico < 200
  geom_vline(xintercept = 200, color = "red1", linetype = "dotdash", lwd = .8)+
  # Básico entre 200-275
  geom_vline(xintercept = 275, color = "blue", linetype = "dotdash", lwd = .8)+
  # Adequado entre 275-325
  geom_vline(xintercept = 325, color = "cyan4", linetype = "dotdash", lwd = .8)
  # Avançado maior que 325
histLP

# Matemática ----
# Camada 1 : Histograma
histMT <- ggplot(mydata, aes(x = MEDIA_9EF_MT) )+
  theme_bw()+
  geom_histogram()+
  #outlier.color = "darkblue"
  # stat_summary(fun.y= mean, geom="point", 
  #              shape=20, size= 2, color="red", fill="red")+
  facet_grid( rows = vars(ano) )

# Camada 2 : labels
histMT <- histMT +
  #scale_color_manual(values = c("brown4","turquoise1","green"))+
    scale_x_continuous(
    name = "Desempenho médio em Matemática",
    limits = c(100,400)) +
  scale_y_continuous(
    name = "Frequência" )

# Camada 3 : linhas horizontais
histMT <- histMT +
  # Abaixo do básico < 225
  geom_vline(xintercept = 225, color = "red1", linetype = "dotdash", lwd = .8)+
  # Básico entre 225-300
  geom_vline(xintercept = 300, color = "blue", linetype = "dotdash", lwd = .8)+
  # Adequado entre 300-350
  geom_vline(xintercept = 350, color = "cyan4", linetype = "dotdash", lwd = .8)
  # Avançado: acima de 350
histMT
```


Observamos que a proficiência tem uma distribuição próxima da normal, com parte significativa das escolas concentradas no nível básico em LP e MT. Poucos são os casos de escolas no nível adequado, e ainda menos no avançado.

É possível observar também um pequeno deslocamento dos casos para a direita de 2013-2019, o que representa um aumento em proficiência das escolas em geral. Apenas com a distribuição do desempenho podemos observamos uma a situação sensível em termos de habilidades escolares. São poucos os casos de escolas com desempenho médio no nível adequado, e nenhuma no nível avançado entre 2013 e 2017.

```{r include=F,message=F}
mydata$ano <- as.character(mydata$ano)
textcol = "grey40"

mydata %>% ggplot(.,aes(x=MEDIA_9EF_LP,fill=ano))+
    geom_histogram(aes(y=..density..))+
    geom_density()+
    theme_bw()+
    scale_y_continuous(name = "densidade")+
    scale_x_continuous(limits = c(150,380), name = "Desempenho em LP")+
    scale_fill_manual(values=rev(c("#d53e4f","#f46d43","#fdae61","#fee08b")))+
    labs(title="Distribuição do desempenho escolar",
         subtitle = "Brasil, 2013-2019")+
    guides(fill=guide_legend(title = "Ano"))+
  
    theme(legend.position="right",legend.direction="vertical",
        legend.title=element_text(colour=textcol),
        legend.margin=margin(grid::unit(0,"cm")),
        legend.text=element_text(colour=textcol,size=7,face="bold"),
        legend.key.height=grid::unit(0.8,"cm"),
        legend.key.width=grid::unit(0.3,"cm"),
        axis.text.x=element_text(size=10,colour=textcol),
        axis.text.y=element_text(vjust=0.2,colour=textcol),
        axis.ticks=element_line(size=0.4),
        plot.background=element_blank(),
        panel.border=element_blank(),
        plot.margin=margin(0.7,0.4,0.1,0.2,"cm"),
        plot.title=element_text(colour=textcol,hjust=0,size=12,face="bold"),
        plot.subtitle = element_text(colour=textcol,hjust=0,size=12))
```

# Indicador de Nível Socioeconômico

O indicador de Nível Socioeconômico (INSE, daqui em diante) é um constructo teórico operacionalizado em uma medida derivada dos itens do questionário socioeconômico dos alunos. Conforme metodologia construída por Alves e Soares (2009, 2011), foi também desenvolvido com base na Teoria de Resposta ao Item (TRI) e está disponibilizado nas bases oficiais do INEP. De acordo com os Alves e Soares (2009), o INSE segue um constructo de medida socioeconômica tradicional no escopo das pesquisas sobre estratificação, e possui correlação forte com outros indicadores como o Índice de Desenvolvimento Humano Municipal (IDHM) e a Renda Domiciliar Per Capita (RDPC) nos municípios. Por meio de uma análise de cluster por método hierárquico, os estudantes foram agrupados em sete níveis ordinais segundo os itens respondidos. Dos sete grupos, derivamos três categorias sintéticas de INSE: i) baixo, que agrega os grupos um e dois; ii) médio, que agrega os grupos três, quatro e cinco e; iii) alto, que agrega os grupos seis e sete.

```{r message=F,include=F}
pltsLP <- list(s13 = NA, s15 = NA, s17 = NA, s19 = NA)
pltsMT <- list(s13 = NA, s15 = NA, s17 = NA, s19 = NA)

for(i in 1:length(ano)){
pltsLP[[i]] <- my_saeb[[i]] %>% ggplot(.,aes(x=INSE,fill=INSE))+
    geom_bar()+
    theme_bw()+
    scale_y_discrete(name = "Frequência")+
    scale_x_discrete(name = "Indicador de Nível Socioeconômico (INSE)")+
    labs(title=ano[i])+
    guides(fill=guide_legend(title = "Ano"))+

      theme(legend.position="right",legend.direction="vertical",
        legend.title=element_text(colour=textcol),
        legend.margin=margin(grid::unit(0,"cm")),
        legend.text=element_text(colour=textcol,size=7,face="bold"),
        legend.key.height=grid::unit(0.8,"cm"),
        legend.key.width=grid::unit(0.3,"cm"),
        axis.text.x=element_text(size=10,colour=textcol),
        axis.text.y=element_text(vjust=0.2,colour=textcol),
        axis.ticks=element_line(size=0.4),
        plot.background=element_blank(),
        panel.border=element_blank(),
        plot.margin=margin(0.7,0.4,0.1,0.2,"cm"),
        plot.title=element_text(colour=textcol,hjust=0,size=12,face="bold"),
        plot.subtitle = element_text(colour=textcol,hjust=0,size=12))
}

plot_grid(pltsLP[[1]], pltsLP[[2]], pltsLP[[3]], pltsLP[[4]],labels="AUTO")
```

# Desempenho por Nível Socioeconômico

Agora veremos a correlação entre nível socioeconômico e desempenho médio. Em primeiro lugar, condicionamos a distribuição do desempenho pelo INSE para verificar o padrão de distanciamento ao longo do tempo.

```{r include=FALSE,message=F}
pltsLP <- list(s13 = NA, s15 = NA, s17 = NA, s19 = NA)
pltsMT <- list(s13 = NA, s15 = NA, s17 = NA, s19 = NA)

for(i in 1:length(ano)){
pltsLP[[i]] <- my_saeb[[i]] %>% ggplot(.,aes(x=MEDIA_9EF_LP,fill=INSE))+
    geom_histogram(aes(y=..density..))+
    geom_density()+
    theme_bw()+
    scale_y_continuous(name = "densidade")+
    scale_x_continuous(limits = c(150,380), name = "Desempenho em LP")+
    labs(title=ano[i])
}

for(i in 1:length(ano)){
pltsMT[[i]] <- my_saeb[[i]] %>% ggplot(.,aes(x=MEDIA_9EF_LP,fill=INSE))+
    geom_histogram(aes(y=..density..))+
    geom_density()+
    theme_bw()+
    scale_y_continuous(name = "densidade")+
    scale_x_continuous(limits = c(150,380), name = "Desempenho em LP")+
    labs(title=ano[i])
}

plot_grid(pltsLP[[1]], pltsLP[[2]], pltsLP[[3]], pltsLP[[4]],labels="AUTO")
plot_grid(pltsMT[[1]], pltsMT[[2]], pltsMT[[3]], pltsMT[[4]],labels="AUTO")

```

Os diagramas representam na caixa central os três intervalos interquartis (IQR’s) da distribuição de proficiência, isto é, os pontos 25%, 50% (mediana) e 75% da distribuição ordenada. As linhas estendidas fora da caixa representam 1,5 do IQR, e os triângulos representam as observações fora da linha, que estão muito distantes da concentração. As linhas horizontais tracejadas representam a interpretação pedagógica dos níveis de proficiência da escala SAEB. A distribuição da proficiência condicionada ao INSE apresenta uma escada da hierarquia constante entre as Escolas, o que indica o padrão de desigualdades persistentes entre os grupos. Somado ao padrão de desigualdades, estão os casos extremos de Escolas com baixo e médio INSE e com desempenho médio abaixo de 200 (LP) e 225 (MT). Essas Escolas estariam contextualizadas no problema discutido pelos teóricos da reprodução (BOURDIEU; PASSERON, 2018), uma vez que têm refletida na proficiência a condição socioeconômica dos estudantes. Segundo a escala de proficiência definida pelo SAEB, estes estudantes estão em uma condição crítica de aprendizado nas disciplinas de referência, sem o domínio de habilidades elementares, e por isso requerem atenção especial (INEP, 2019). Também chama a atenção as Escolas com resultados acima de 325 (LP) e 350 (MT) nessas mesmas categorias. Essas configuram casos particulares de Escolas que possivelmente obtêm êxito no controle do background socioeconômico dos estudantes, oferecendo condições institucionais de aprendizado independente da origem social, tratados pela literatura sobre Eficácia escolar.
Para comparar as distribuições de proficiência entre os grupos de NSE e mensurar a distância entre estes, calculamos a distância entre mediana da proficiência de LP e MT do grupo com INSE baixo em relação aos grupos de médio e alto NSE, representados na tabela abaixo.

Para analisar com maior detalhe as distâncias entre os grupos de nível socioeconômico nesse período, fizemos uma análise de diferença por pontos percentis da distribuição de proficiência condicionada ao INSE. Calculamos a diferença em cada ponto percentil entre os grupos de INSE alto e médio em relação ao baixo. Com isso, podemos observar não apenas as distâncias em média ou em mediana, mas em cada ponto percentil da distribuição de proficiência de cada grupo marcado pelo INSE.

```{r message=FALSE, include=T}
# Distribuição pontos percentis (pp) da distribuição de proficiência ==========

pp_LP <- list(
                     s13 = data.frame(Baixo = rep(NA,100), Médio = rep(NA,100), Alto = rep(NA,100)),
                     s15 = data.frame(Baixo = rep(NA,100), Médio = rep(NA,100), Alto = rep(NA,100)),
                     s17 = data.frame(Baixo = rep(NA,100), Médio = rep(NA,100), Alto = rep(NA,100)),
                     s19 = data.frame(Baixo = rep(NA,100), Médio = rep(NA,100), Alto = rep(NA,100)))

pp_MT <- list(
  s13 = data.frame(Baixo = rep(NA,100), Médio = rep(NA,100), Alto = rep(NA,100)),
  s15 = data.frame(Baixo = rep(NA,100), Médio = rep(NA,100), Alto = rep(NA,100)),
  s17 = data.frame(Baixo = rep(NA,100), Médio = rep(NA,100), Alto = rep(NA,100)),
  s19 = data.frame(Baixo = rep(NA,100), Médio = rep(NA,100), Alto = rep(NA,100)))

for(i in 1:length(ano)) { 
  baixo <- subset(my_saeb[[i]], subset = (INSE == "Baixo"))
  medio <- subset(my_saeb[[i]], subset = (INSE == "Medio"))
  alto  <- subset(my_saeb[[i]], subset = (INSE == "Alto"))
  # Lingua Portuguesa
  pp_LP[[i]][,1] <- quantile(baixo$MEDIA_9EF_LP, probs = seq(0.01,1,0.01))
  pp_LP[[i]][,2] <- quantile(medio$MEDIA_9EF_LP, probs = seq(0.01,1,0.01))
  pp_LP[[i]][,3] <- quantile(alto$MEDIA_9EF_LP,  probs = seq(0.01,1,0.01))
  # Matematica
  pp_MT[[i]][,1] <- quantile(baixo$MEDIA_9EF_MT, probs = seq(0.01,1,0.01))
  pp_MT[[i]][,2] <- quantile(medio$MEDIA_9EF_MT, probs = seq(0.01,1,0.01))
  pp_MT[[i]][,3] <- quantile(alto$MEDIA_9EF_MT,  probs = seq(0.01,1,0.01))
}

qqdif <- list(
              s13 = data.frame(baixo_medioLP = rep(NA, 100), baixo_altoLP = rep(NA, 100),baixo_medioMT = rep(NA, 100), baixo_altoMT = rep(NA, 100)),
              s15 = data.frame(baixo_medioLP = rep(NA, 100), baixo_altoLP = rep(NA, 100),baixo_medioMT = rep(NA, 100), baixo_altoMT = rep(NA, 100)),
              s17 = data.frame(baixo_medioLP = rep(NA, 100), baixo_altoLP = rep(NA, 100),baixo_medioMT = rep(NA, 100), baixo_altoMT = rep(NA, 100)),
              s19 = data.frame(baixo_medioLP = rep(NA, 100), baixo_altoLP = rep(NA, 100),baixo_medioMT = rep(NA, 100), baixo_altoMT = rep(NA, 100)))
pltsLP <- list(s13 = NA, s15 = NA, s17 = NA, s19 = NA)
pltsMT <- list(s13 = NA, s15 = NA, s17 = NA, s19 = NA)


for(i in 1:length(ano)) {
qqdif[[i]][,1] <- pp_LP[[i]][,2] - pp_LP[[i]][,1]

qqdif[[i]][,2] <- pp_LP[[i]][,3] - pp_LP[[i]][,1]

qqdif[[i]][,3] <- pp_MT[[i]][,2] - pp_MT[[i]][,1]

qqdif[[i]][,4] <- pp_MT[[i]][,3] - pp_MT[[i]][,1]}

for(i in 1:length(ano)) {
pltsLP[[i]] <- qqdif[[i]] %>% ggplot(aes(x = seq(1,100,1)))+
  theme_bw()+
  geom_point(y = qqdif[[i]]$baixo_medioLP, colour = "blue")+
  geom_point(y = qqdif[[i]]$baixo_altoLP, colour = "red")+
  geom_hline(yintercept = 0, colour = "steelblue")+
  scale_y_continuous(limits = c(-20,60), name = "diferença")+
  scale_x_continuous(breaks = seq(0,100,20), name = "quantis")+
  labs(title=ano[i])
}

for(i in 1:length(ano)) {
  pltsMT[[i]] <- qqdif[[i]] %>% ggplot(aes(x = seq(1,100,1)))+
    theme_bw()+
    geom_point(y = qqdif[[i]]$baixo_medioMT, colour = "blue")+
    geom_point(y = qqdif[[i]]$baixo_altoMT, colour = "red")+
    geom_hline(yintercept = 0, colour = "steelblue")+
    scale_y_continuous(limits = c(-20,60), name = "diferença")+
    scale_x_continuous(breaks = seq(0,100,20), name = "quantis")+
    labs(title=ano[i])
    
  }

plot_grid(pltsLP[[1]], pltsLP[[2]], pltsLP[[3]], pltsLP[[4]],labels="AUTO")
```

