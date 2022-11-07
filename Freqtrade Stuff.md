[Freqtrade Stuff](https://brookmiles.github.io/freqtrade-stuff/)
 Orignal from Freqtrade [Stuff](https://brookmiles.github.io/freqtrade-stuff/)

**Les pièges du backtesting**

2021-04-12 

Le backtesting est important pour comprendre les performances et le comportement de votre stratégie, mais il existe un certain nombre de problèmes qui peuvent faire qu'un backtest produise des résultats bien meilleurs que ceux qui sont probables une fois que la stratégie est exécutée en direct (ou dry-run).

Ces informations s'appliquent également à la fonction Hyperopt, qui traite les stratégies de la même manière que le backtesting.

Pour commencer, assurez-vous de vous familiariser avec la [documentation officielle sur le backtesting](https://www.freqtrade.io/en/stable/backtesting/#assumptions-made-by-backtesting) et ses hypothèses.

**Principes de base**

- L'avenir sera différent du passé

Les tests de backtesting utilisent des données historiques, donc même si votre stratégie évite tous les autres problèmes, il n'y a aucune garantie qu'elle réussira dans le futur, même si elle a réussi sur des données passées.

- Délais d'exécution des transactions (Trade Timeouts)

Lors d'un backtesting, tous vos ordres sont magiquement exécutés au prix exact que vous souhaitiez. Cela ne se produit pas dans la vie réelle, et si vous placez des ordres à cours limité, il est probable qu'un certain pourcentage d'entre eux ne soit pas exécuté et que le délai d'exécution soit dépassé, ce qui vous empêche d'entrer ou de sortir d'une transaction. La probabilité que cela se produise dépend du volume de la paire que vous négociez, du moment où votre stratégie passe des ordres, de la hausse ou de la baisse du marché, du côté de l'écart entre les cours acheteur et vendeur où vous placez vos ordres, etc...

- Dérapage (Slippage)

Quel que soit le type d'ordre que vous utilisez, le marché continue de bouger entre le moment où votre stratégie décide de passer un ordre et celui où cet ordre est effectivement passé et, espérons-le, exécuté sur le marché. En particulier lorsque vous utilisez des ordres au marché, cela entraîne un glissement de prix et il est probable que vous obteniez un prix un peu moins bon qu'au moment où la stratégie a décidé de passer un ordre. Le degré de détérioration dépendra des conditions exactes du marché, du spread, du volume, de la taille de votre ordre, de la direction du prix, etc...

Reportez-vous aux sections [Prix utilisés pour les ordres](https://www.freqtrade.io/en/stable/configuration/#prices-used-for-orders) et [Prix des ordres au marché](https://www.freqtrade.io/en/stable/configuration/#market-order-pricing) dans la documentation sur la configuration pour plus de détails.



Points à surveiller lors du backtesting :

- Un grand nombre de transactions et un faible profit moyen. Si votre stratégie est rentable dans le backtesting sur la base de nombreuses transactions avec de très petits profits, il est très probable que ces profits se transforment en pertes dans le réel en raison du slippage, en particulier sur les paires à faible volume. Soyez prudent si votre profit moyen est inférieur à 0,5 %, et méfiez-vous de tout changement qui augmente votre profit total mais diminue votre profit moyen.


**Lecture des bougies du futur**

Une chose à laquelle il peut être difficile de s'habituer lors d'un backtesting est que les signaux sont générés à partir de la totalité de la plage de temps en une seule fois. Le robot ne commence pas à la première bougie et traite chaque bougie dans l'ordre.

Les fonctions populate\_indicators, populate\_buy\_trend et populate\_sell\_trend sont appelées une fois pour une paire donnée et génèrent tous les signaux pour toute la durée. Cela signifie que la "fin" de la trame de données est toujours la dernière bougie de la période de backtesting, et non la bougie "la plus récente" comme ce serait le cas lors d'une exécution en direct ou dry.

Lors de la création des signaux pour une bougie donnée, il est donc facile de lire accidentellement les bougies qui suivent dans le Dataframe, en "voyant dans le futur". En mode dry ou en direct, le futur n'est pas disponible.

Lisez la documentation officielle sur [les erreurs courantes commises lors du développement de stratégies](https://www.freqtrade.io/en/stable/strategy-customization/#common-mistakes-when-developing-strategies) pour obtenir des détails sur les choses à éviter.
**


**Trailing Stops**

"Je continue à resserrer les trailing stoploss et les profits du backtest augmentent. Cela ne peut pas être vrai."

Une anomalie de backtesting qui semble produire des résultats incroyables, on pourrait même dire invraisemblables, est l'utilisation de très petits trailing stops.

Voici un exemple de stratégie qui exploite cette caractéristique. Elle n'utilise aucun signal, et achète simplement sur chaque bougie sur une échelle de temps d'une heure. Les ventes ont lieu lorsque le retour sur investissement minimum de 1 % est atteint, ou lorsque le trailing stops de 0,1 % est atteint.

```
from freqtrade.strategy.interface import IStrategy
from typing import Dict, List
from pandas import DataFrame

class Magic_Trailing_Stoploss(IStrategy):

    minimal_roi = { "0": 0.01 }

    stoploss = -0.01

    trailing_stop = True
    trailing_stop_positive = 0.001

    timeframe = '1h'

    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        return dataframe

    def populate_buy_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        dataframe['buy'] = 1
        return dataframe

    def populate_sell_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        dataframe['sell'] = 0
        return dataframe

```



Maintenant nous faisons un backtest...

![40E0F5B7.tmp](Aspose.Words.cceefcef-1a02-4cdc-afba-69f73efc1fd5.001.png)

En seulement 11 jours, nous sommes passés de 2 500 $ à 65 150 $, soit un bénéfice de 2506 % ! Incroyable !

Malheureusement, cette stratégie ne produira pas de résultats similaires lorsqu'elle sera exécutée en direct.

Tout d'abord, jetez un coup d'œil à la durée moyenne des gagnants et des perdants. Durée moyenne des gagnants et des perdants. 1 minute, et 7 minutes respectivement. Lorsque vous négociez des bougies d'une heure, c'est un signe que vous exploitez le biais de comportement du backtesting.



L'utilisation de [plot-dataframe](https://www.freqtrade.io/en/latest/plotting/) peut aider à illustrer ce qui se passe :

![DDF15FDD.tmp](Aspose.Words.cceefcef-1a02-4cdc-afba-69f73efc1fd5.002.png)

La plupart des bougies montrent que la transaction a été ouverte et fermée dans la même bougie, ce que le backtesting considère comme 0 minute, ou sur la bougie juste suivante.

**Que se passe-t-il ?**

Lors d'un backtesting sur n'importe quelle période, Freqtrade ne dispose que des données des bougies pour travailler. Il ne sait pas comment le prix a évolué pendant la bougie. Avec des bougies de 1m ou 5m, c'est moins un problème (bien qu'il existe toujours), mais avec des délais plus longs, comme 1h, 4h, etc... Freqtrade doit faire quelques hypothèses majeures sur l'ordre des événements.

Comme mentionné dans [la documentation officielle sur le backtesting](https://www.freqtrade.io/en/stable/backtesting/#assumptions-made-by-backtesting), lorsque Freqtrade calcule le trailing stoploss, il déplace d'abord le prix vers le haut, déplace le trailing stop vers le haut pour correspondre en conséquence, puis déplace le prix vers le bas, déclenchant éventuellement le nouveau prix du trailing stop.

Le backtesting avec des trailing stops extrêmement serrés vous donne essentiellement une transaction parfaite sur une bougie, en vendant juste en dessous du haut de la bougie, presque à chaque fois.

Plus l'échelle de temps utilisée est longue, moins cela est réaliste. Avec un petit stoploss de suivi, il est exceptionnellement improbable que le prix passe directement de l'ouverture au sommet sans diminuer suffisamment pour déclencher le stoploss. Si votre stoploss est inférieur au spread de la paire que vous négociez, il sera probablement atteint lors de la prochaine transaction de vente. Plus l'horizon temporel est long, plus votre stop suiveur, ou même votre stoploss par défaut, a de chances d'être déclenché avant d'atteindre le sommet puis de baisser.

**ROI**

Si vous avez défini un ROI raisonnablement serré, il se peut que vous voyiez le ROI atteint au sommet exacte de la bougie, mais le même problème de trailing stop s'applique. Plus la bougie est longue, moins il est probable que le prix atteigne le ROI avant de déclencher votre trailing stop.
**


**Ce qu'il faut surveiller**

Les éléments à surveiller lors du développement/backtest de votre stratégie qui pourraient être un indice que vous exploitez accidentellement ce comportement :

- La durée moyenne des transactions est inférieure à l'horizon temporel de votre bougie, peut-être seulement quelques minutes si vous utilisez des bougies de 1 heure.

- Lorsqu'elles sont tracées, les transactions se clôturent régulièrement sur la même bougie que celle où elles ont été ouvertes.

- Votre trailing stop est inférieur ou à peine supérieur à l'écart sur les paires que vous négociez. Les spreads typiques vont de 0,1 à 0,5 % sur les pièces à fort volume, et sont souvent beaucoup plus élevés sur les pièces à faible volume.

- Les profits augmentent à mesure que vous réduisez la taille de votre stop suiveur.

**Contournement du Trailing Stop**

Une solution que vous pouvez utiliser pour vous assurer que vous obtenez des résultats de backtest plus réalistes est de backtester votre stratégie 1h à 5m ou 1m, en utilisant une paire informative à 1h pour générer des signaux de trade. Les signaux fonctionneront sur la même échelle de temps d'une heure qu'en temps réel, et l'exécution de la stratégie à l'aide de données de bougies de 5m ou d'1m simulera le mouvement des prix au cours de cette heure, fournissant un comportement de stoploss et de retour sur investissement plus réaliste.

Malheureusement, cette méthode est plus lente que le backtesting sur une heure et complique quelque peu le code. Mais si vous prévoyez d'utiliser un stoploss ou un ROI suiveur, vous voulez probablement savoir que les résultats de votre backtest ne sont pas des mensonges complets.

Voici une stratégie simple qui utilise un simple croisement d'EMA pour générer des signaux d'achat :

- [EMA_Trailing_Stoploss](https://github.com/brookmiles/freqtrade-stuff/blob/main/strategies/examples/EMA_Trailing_Stoploss.py)

```
from freqtrade.strategy import IStrategy
from typing import Dict, List
from pandas import DataFrame
import talib.abstract as ta
import freqtrade.vendor.qtpylib.indicators as qtpylib

class EMA_Trailing_Stoploss(IStrategy):

    minimal_roi = { "0": 0.01 }

    stoploss = -0.01

    trailing_stop = True
    trailing_stop_positive = 0.001

    timeframe = '1h'

    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:

        dataframe['ema3'] = ta.EMA(dataframe, timeperiod=3)
        dataframe['ema5'] = ta.EMA(dataframe, timeperiod=5)
        dataframe['go_long'] = qtpylib.crossed_above(dataframe['ema3'], dataframe['ema5']).astype('int')

        return dataframe

    def populate_buy_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        dataframe.loc[
            qtpylib.crossed_above(dataframe['go_long'], 0)
        ,
        'buy'] = 1

        return dataframe

    def populate_sell_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        dataframe['sell'] = 0
        return dataframe

```

Le backtesting montre un bénéfice très respectable de 11%, mais c'est une illusion :

![163D1AF3.tmp](Aspose.Words.cceefcef-1a02-4cdc-afba-69f73efc1fd5.003.png)

Et la même stratégie après l'avoir convertie pour être backtestée à 5m ou 1m :

- [EMA_Trailing_Stoploss_LessMagic](https://github.com/brookmiles/freqtrade-stuff/blob/main/strategies/examples/EMA_Trailing_Stoploss_LessMagic.py)
- Changer le cadre temporel à 5m et ajouter informative\_timeframe
- Déplacer les indicateurs de populate\_indicators à une nouvelle fonction appelée do\_indicators
- populate\_indicators vérifie l'horizon temporel et le mode d'exécution, et exécute do\_indicators directement en temps réel, ou l'exécute en utilisant les paires informatives sur une heure, puis fusionne les résultats dans l'horizon temporel principal lors du backtesting.
- Mettez toute votre création de signaux dans do\_indicators, et lisez seulement les signaux d'achat/vente de populate\_buy\_trend / populate\_sell\_trend.

```
from freqtrade.strategy import IStrategy, merge_informative_pair
from typing import Dict, List
from pandas import DataFrame
import talib.abstract as ta
import freqtrade.vendor.qtpylib.indicators as qtpylib
from freqtrade.exchange import timeframe_to_minutes

class EMA_Trailing_Stoploss_LessMagic(IStrategy):

    minimal_roi = { "0": 0.01 }

    stoploss = -0.01

    trailing_stop = True
    trailing_stop_positive = 0.001

    timeframe = '5m'
    informative_timeframe = '1h'

    def informative_pairs(self):
        pairs = self.dp.current_whitelist()
        informative_pairs = [(pair, self.informative_timeframe) for pair in pairs]
        return informative_pairs

    def do_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        dataframe['ema3'] = ta.EMA(dataframe, timeperiod=3)
        dataframe['ema5'] = ta.EMA(dataframe, timeperiod=5)
        dataframe['go_long'] = qtpylib.crossed_above(dataframe['ema3'], dataframe['ema5']).astype('int')
        return dataframe

    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:

        if self.config['runmode'].value in ('backtest', 'hyperopt'):
            assert (timeframe_to_minutes(self.timeframe) <= 5), "Backtest this strategy in 5m or 1m timeframe."

        if self.timeframe == self.informative_timeframe:
            dataframe = self.do_indicators(dataframe, metadata)
        else:
            if not self.dp:
                return dataframe

            informative = self.dp.get_pair_dataframe(pair=metadata['pair'], timeframe=self.informative_timeframe)

            informative = self.do_indicators(informative.copy(), metadata)

            dataframe = merge_informative_pair(dataframe, informative, self.timeframe, self.informative_timeframe, ffill=True)
            skip_columns = [(s + "_" + self.informative_timeframe) for s in ['date', 'open', 'high', 'low', 'close', 'volume']]
            dataframe.rename(columns=lambda s: s.replace("_{}".format(self.informative_timeframe), "") if (not s in skip_columns) else s, inplace=True)

        return dataframe

    def populate_buy_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        dataframe.loc[
            qtpylib.crossed_above(dataframe['go_long'], 0)
        ,
        'buy'] = 1

        return dataframe

    def populate_sell_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        dataframe['sell'] = 0
        return dataframe

```

Le backtesting montre maintenant que tous les profits ont disparu, ce qui est beaucoup plus proche de ce qui se passerait en direct, et sera encore plus bas une fois que le spread, le slippage, les ordres non remplis, etc... seront pris en compte.

![5E478479.tmp](Aspose.Words.cceefcef-1a02-4cdc-afba-69f73efc1fd5.004.png)

