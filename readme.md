http://dev-poc-conferences.philharmoniedeparis.fr/api/recordings?filters[$and][0][stream][$ne]=&populate=*&pagination[pageSize]=200&pagination[page]=1

# Sous-sous parties
https://pad.philharmoniedeparis.fr/pad/doc/CIMU/0771412/forum-musical-la-flute-en-occident

##
http://dev-poc-conferences.philharmoniedeparis.fr/api/recordings?filters[$and][0][stream][$ne]=&populate=*&pagination[pageSize]=200&filters[$and][1][syracuse_id]=771412

* LLM ne lit que des audios format unifrmisé ffmpeg
https://github.com/m-bain/whisperX
* Whisper pour la transcription
* WordAlignement : alignement
* Un modèle pour les speakers

* Il manque des métadonnées sur des groupes de conférences. Pourrait-on les récupérer ?
    * https://pad.philharmoniedeparis.fr/search.aspx?SC=MEDIA&QUERY=Identifier_idx%3a%220729685%22&QUERY_LABEL=Document&DETAIL_MODE=true#/Detail/(query:(Id:0,Index:1,NBResults:1,Page:0,PageRange:3,ResultSize:-1,SearchQuery:(InitialSearch:!t,Page:0,QueryString:'Identifier_idx:%220729685%22',ResultSize:-1,ScenarioCode:MEDIA,SearchContext:0,SearchLabel:Document)))
* Quelques sont les significations des types de notices ?
* Il y a un troisième niveau de hiérarchie (MPV3), a quoi correspond il, est-ce que ça a un intérêt pour nous et pourrait on le récupérer aussi ?
* Il y a des éléments qui sont des interprétations entières d'oeuvres et ne sont pas des conférences a proprement parler. Les écarter (et éventuellement prévoir une rubrique 'on a joué' ? Sachant qu'on ne vas pas chercher les extraits qui ne sont pas inventoriés)