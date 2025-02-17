1. После проверки конфигурации выполните команду:
   ```
   terraform plan
   ```

   В терминале будет выведен список ресурсов с параметрами. Это проверочный этап: ресурсы не будут созданы. Если в конфигурации есть ошибки, Terraform на них укажет.

     {% note alert %}
  
     Все созданные с помощью Terraform ресурсы тарифицируются. Внимательно проверьте план.
  
     {% endnote %}

1. Чтобы создать ресурсы выполните команду:
   ```
   terraform apply
   ```
   
1. Подтвердите создание ресурсов: введите в терминал слово `yes` и нажмите **Enter**.

   Terraform создаст все требуемые ресурсы, а в терминале будут указаны IP-адреса созданных ВМ. Проверить появление ресурсов и их настройки можно в [консоли управления]({{ link-console-main }}).
