# Salt DevKit

Локальная разработка формул SaltStack используя Vagrant

## Использование

* Склонируйте git репозиторий локально
* Создайте копию проекта `template`, например можно назвать ее `work` (папка `work` добавлена в `.gitignore`) при этом вы получите пустой проект, без каких либо стейтов или палларов.
* При желании можно скопировать файлы из папки `demo`, в качестве пример там имеется простейший стейт `common.packages` который устанавливает несколько пакетов.
* Либо разместите собственные файлы конфигурации для мастера и миньонов, создайте pillar-ы и state файлы отредактируйте `config.yaml` при необходимости, например, для подключения дополнительных папок.

## Ссылки

* <https://joachim8675309.medium.com/salt-devkit-developing-formulas-e8b500d6b970>
* <https://joachim8675309.medium.com/salt-devkit-with-external-formulas-9e38d8b90cd7>
* <https://docs.saltstack.com/en/getstarted/fundamentals/>
  * <https://github.com/UtahDave/salt-vagrant-demo>
