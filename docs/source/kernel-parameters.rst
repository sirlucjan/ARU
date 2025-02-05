.. ARU (c) 2018 - 2022, Pavel Priluckiy, Vasiliy Stelmachenok and contributors

   ARU is licensed under a
   Creative Commons Attribution-ShareAlike 4.0 International License.

   You should have received a copy of the license along with this
   work. If not, see <https://creativecommons.org/licenses/by-sa/4.0/>.

.. _kernel-parameters:

**************************
Настройка параметров ядра
**************************

.. index:: bootloader, patch-off, grub, grub-customizer, mitigations-off
.. _update-bootloader-parameters:

=====================================================
Обновление загрузчика и отключение ненужных заплаток
=====================================================

По умолчанию в ядре Linux включено довольно много исправлений безопасности,
которые однако существенно снижают производительность процессора. Вы можете их
отключить через редактирование параметров загрузчика. Рассмотрим на примере
GRUB:

``sudo nano /etc/default/grub`` # Редактируем настройки вручную или через grub-customizer как на изображении:

.. image:: images/kernel-parameters-1.png

::

  GRUB_CMDLINE_LINUX_DEFAULT="quiet splash rootfstype=btrfs lpj=3499912 raid=noautodetect elevator=noop mitigations=off preempt=none nowatchdog audit=0 page_alloc.shuffle=1 split_lock_detect=off"


``sudo grub-mkconfig -o /boot/grub/grub.cfg`` # Обновляем загрузчик, можно так
же сделать через grub-customizer, добавить и прожать, затем сохранить на 2 и 1
вкладке.

.. index:: settings, mitigations-off
.. _explanations:

--------------
Разъяснения
--------------

``lpj=`` - Уникальный параметр для каждой системы. Его значение автоматически
определяется во время загрузки, что довольно трудоемко, поэтому лучше задать
вручную. Определить ваше значение для lpj можно через следующую команду: ``sudo
dmesg | grep "lpj="``

``mitigations=off`` - Непосредственно отключает все заплатки безопасности ядра
(включая Spectre и Meltdown). Подробнее об этом написано `здесь
<https://linuxreviews.org/HOWTO_make_Linux_run_blazing_fast_(again)_on_Intel_CPUs>`_.

``raid=noautodetect`` - Отключает проверку на RAID во время загрузки. Если вы
его используете - **НЕ** прописывайте данный параметр.

``rootfstype=btrfs`` - Здесь указываем название файловой системы в которой у
вас отформатирован корень.

``elevator=noop`` - Указывает для всех дисков планировщик ввода NONE.
Полезно, если у вас NVMe. **Не использовать если у вас жесткий диск**.

``nowatchdog`` - Отключает сторожевые таймеры. Позволяет избавиться от заиканий
в онлайн играх.

``page_alloc.shuffle=1`` - Этот параметр рандомизирует свободные списки распределителя страниц.
Улучшает производительность при работе с ОЗУ с очень быстрыми накопителями (NVMe, Optane).
Подробнее `тут
<https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=e900a918b0984ec8f2eb150b8477a47b75d17692>`__.

``split_lock_detect=off`` - Отключаем раздельные блокировки шины
памяти. Одна инструкция с раздельной блокировкой может занимать шину
памяти в течение примерно 1 000 тактов, что может приводить к
кратковременным зависаниям системы.

.. vim:set textwidth=70:
