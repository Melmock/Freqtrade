# **Par où commencer avec Freqtrade**

Document original [ici](https://brookmiles.github.io/freqtrade-stuff/2021/04/20/where-to-start-with-freqtrade/)

20-04-2021

Bonjour, je suis nouveau. Avez-vous des conseils à donner aux débutants ?

Je ne suis pas un expert, et rien de ce qui suit **n'est un conseil financier**. Ce sont mes opinions basées sur mon expérience jusqu'à présent.

**Conseils financiers**

Le trading est risqué et le trading de crypto est **particulièrement risqué**. Ne négociez que ce que vous êtes prêt à perdre.

Ma première priorité est de préserver mon capital.

Le trading de crypto-monnaies n'est pas pour vous. Vous pouvez envisager un portefeuille équilibré de FNB (Fonds négociés en bourse), des obligations, des CPG ou un compte d'épargne à taux élevé. Parlez-en à un conseiller financier qualifié

Je veux faire des profits fous, et je suis prêt à risquer de perdre tout ce que j'ai investi pour avoir la chance de les obtenir.

Continuez si vous avez bien compris les conséquences.

**Principes de base**

Il y a en gros 3 domaines différents avec lesquels vous devez vous familiariser pour trader avec Freqtrade.

1. La terminologie générale du trading et l'analyse technique.
2. Une base de compétences techniques et des connaissances de base en programmation. La capacité de lire et de comprendre le code Python.
3. Comment utiliser Freqtrade spécifiquement, ses caractéristiques et ses limites.

**1. Trading en général**

Vous devez avoir une compréhension de base du fonctionnement du trading en général. Assurez-vous de comprendre les termes : _market order_ (ordre de marché), _limit order_ (ordre à cours limité) et la différence entre eux, _bid-ask spread (l'écart entre l'offre et la demande)_, _volume_, _slippage_, le fonctionnement du carnet d'ordres (_order book_), le trading au comptant par rapport au trading sur marge (_spot_ vs. _margin_ trading), le fonctionnement d'un ordre _stop-loss order_.

Toutes ces informations sont communes au trading classique et au trading de crypto-monnaies, cherchez-les sur Google ou [Investopedia](https://www.investopedia.com/).

Une fois que vous avez couvert les bases du trading, vous voudrez creuser plus profondément dans l'analyse technique, qui consiste à examiner les données historiques sur les prix et à déterminer les modèles qui devraient se répéter à l'avenir avec suffisamment de confiance pour informer vos décisions de trading. L'analyse technique utilise généralement des modèles de graphiques ou des indicateurs techniques.

Elle s'oppose à l'analyse fondamentale, qui se concentre sur l'examen des performances ou de la valeur réelle des entreprises ou des actifs négociés, des nouvelles, de l'opinion publique, etc…

Les stratégies Freqtrade se concentrent généralement entièrement sur l'analyse technique.

- [Guide vers analyse technique](https://www.investopedia.com/terms/t/technical-analysis-of-stocks-and-trends.asp)

Familiarisez-vous avec les indicateurs de base tels que les moyennes mobiles, le MACD, les bandes de Bollinger et le RSI.

Si votre bourse permet le paper-trading (simulé, sans argent réel), choisissez une crypto-monnaie pour vous entraîner. Configurez un graphique sur votre bourse ou sur TradingView avec quelques indicateurs qui vous intéressent, et effectuez quelques transactions sur papier en fonction de ces indicateurs.

Assurez-vous de lire non seulement ce que les indicateurs montrent directement, mais aussi comment ils sont généralement utilisés par les traders, et dans quel but. Certains surveillent les tendances, d'autres peuvent être utilisés comme signaux d'achat ou de vente, etc...

- [Comment combiner les indicateurs de trading](https://youtu.be/QdbKApfwF-g)

**2. Compétences en programmation**

Je ne recommande pas d'utiliser Freqtrade sans avoir au moins une compréhension de base des principes fondamentaux de la programmation et du langage Python.

Freqtrade n'est pas livré avec une stratégie "par défaut" qui rapporte de l'argent. Vous devez évaluer chaque stratégie que vous avez l'intention d'utiliser pour vous assurer qu'elle fait au moins quelque chose d'un tant soit peu sensé et, dans le pire des cas, qu'elle n'est pas activement malveillante.

Si vous n'êtes pas en mesure d'examiner au moins une stratégie afin de repérer les erreurs évidentes ou le code potentiellement malveillant, vous serez à la merci de celui qui vous a fourni la stratégie.

D'une manière plus générale, les performances des stratégies varient en fonction de la bourse sur laquelle elles sont exécutées, du timing, des conditions actuelles du marché, des paires de négociation utilisées, de l'horizon temporel utilisé, de la configuration spécifique de votre robot freqtrade, etc... Vous devrez peut-être apporter des modifications de base au code de la stratégie afin de la rendre plus adaptée à votre situation spécifique.

- [Python pour les débutants](https://www.python.org/about/gettingstarted/)

Si vous n'êtes pas à l'aise ou intéressé par l'apprentissage des bases de Python, Freqtrade n'est peut-être pas la meilleure solution pour vous.

**3. Particularités de Freqtrade**

Quelques points essentiels :

- Freqtrade ne gagne pas automatiquement de l'argent. Il n'y a pas de stratégie "par défaut" que vous pouvez activer pour gagner de l'argent. Les stratégies sont écrites en Python par vous, ou partagées par d'autres utilisateurs.
- Faire du profit en négociant est un défi, qui prend du temps, et qui n'est pas du tout garanti. Il en va de même pour le trading automatisé, bien qu'idéalement il devienne moins chronophage lorsque (et si) vous trouvez une stratégie et une configuration qui fonctionnent pour vous à long terme et qui peuvent être exécutées avec un minimum de gestion active.

Freqtrade ne prend en charge que le trading au comptant (spot trading). Vous ne pouvez pas l'utiliser pour effectuer des transactions à découvert (short trades), pour négocier en utilisant une marge (margin), pour négocier des contrats d'options (options contracts) ou des contrats à terme (futures).

Freqtrade ne prend pas en charge le cumul de positions. Vous ne pouvez pas utiliser Freqtrade pour acheter progressivement plus d'une pièce. Une fois qu'une transaction est ouverte, elle doit être vendue avant qu'une autre transaction puisse être effectuée pour la même pièce.

**4. Démarrage**

1. Installez Freqtrade en suivant les instructions du [site officiel](https://www.freqtrade.io/), soit en utilisant Docker, soit directement sur votre machine.
2. Récupérez les stratégies gratuites fournies dans un autre dépôt github que le programme freqtrade : [freqtrade-strategies](https://github.com/freqtrade/freqtrade-strategies). Certaines des stratégies rassemblées sous user\_data/strategies/berlinguyinca/ en particulier sont d'excellents points de départ ou références pour développer vos propres stratégies, et seront souvent mentionnées par d'autres utilisateurs de Freqtrade.
3. Téléchargez les données que vous devrez utiliser pour le backtesting en utilisant la commande [download-data](https://www.freqtrade.io/en/stable/data-download/).
4. Reportez-vous à la documentation sur la [configuration](https://www.freqtrade.io/en/stable/configuration/), car il sera nécessaire d'apporter des modifications à votre configuration pour pouvoir faire le reste.
5. Lancez les stratégies de backtesting à l'aide de la commande [backtesting](https://www.freqtrade.io/en/stable/backtesting/).
6. Une fois que vous avez exécuté un ou deux backtests, utilisez la fonctionnalité [Plotting](https://www.freqtrade.io/en/stable/plotting/) pour visualiser les résultats de vos backtests. Cela peut être incroyablement utile pour comprendre ce que votre stratégie fait réellement. Essayez à la fois les commandes plot-dataframe et plot-profit.
7. Essayez de développer votre propre stratégie.
8. Avant de mettre de l'argent réel en jeu, ou même de configurer vos clés d'échange, choisissez une stratégie à exécuter en mode de fonctionnement à sec, qui simulera des transactions en temps réel à l'aide des données publiques fournies par votre bourse.
9. Configurez Telegram pour surveiller le robot pendant vos essais.

Utilisez d'abord la fonction de recherche située en haut du [site officiel de Freqtrade](https://www.freqtrade.io/en/stable/) lorsque vous avez une question sur une commande, un paramètre de configuration ou un sujet spécifique. Si vous ne trouvez pas ce que vous cherchez, vous pouvez demander de l'aide sur [Discord ou Slack](https://www.freqtrade.io/en/stable/#help-discord-slack).
