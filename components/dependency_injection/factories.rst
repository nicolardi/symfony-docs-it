.. index::
   single: DependencyInjection; Factory

Usare un factory per creare servizi
===================================

Il contenitore di servizi di Symfony2 fornisce un modo potente per controllare la
creazione di oggetti, consentendo di specificare parametri da passare al costruttore,
così come chiamate a metodi e impostazioni di parametri. A volte, tuttavia, non fornisce
tutto ciò che è necessario per costruire gli oggetti.
Per tali situazioni, si può usare un factory per creare oggetti e dire al contenitore di
servizi di richiamare un metodo del factory, invece che istanziare direttamente
l'oggetto.

Si supponga di avere un factory che configura e restituisce un nuovo oggetto
``NewsletterManager``::

    class NewsletterFactory
    {
        public function get()
        {
            $newsletterManager = new NewsletterManager();

            // ...

            return $newsletterManager;
        }
    }

Per rendere l'oggetto ``NewsletterManager`` disponibile come servizio, si può configurare 
il contenitore di servizi per usare la classe factory
``NewsletterFactory``:

.. configuration-block::

    .. code-block:: yaml

        parameters:
            # ...
            newsletter_manager.class: NewsletterManager
            newsletter_factory.class: NewsletterFactory
        services:
            newsletter_manager:
                class:          "%newsletter_manager.class%"
                factory_class:  "%newsletter_factory.class%"
                factory_method: get

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

            <parameters>
                <!-- ... -->
                <parameter key="newsletter_manager.class">NewsletterManager</parameter>
                <parameter key="newsletter_factory.class">NewsletterFactory</parameter>
            </parameters>

            <services>
                <service
                    id="newsletter_manager"
                    class="%newsletter_manager.class%"
                    factory-class="%newsletter_factory.class%"
                    factory-method="get" />
            </services>
        </services>

    .. code-block:: php

        use Symfony\Component\DependencyInjection\Definition;

        // ...
        $container->setParameter('newsletter_manager.class', 'NewsletterManager');
        $container->setParameter('newsletter_factory.class', 'NewsletterFactory');

        $definition = new Definition('%newsletter_manager.class%');
        $definition->setFactoryClass('%newsletter_factory.class%');
        $definition->setFactoryMethod('get');

        $container->setDefinition('newsletter_manager', $definition);

Quando si specificare la classe da usare come factory (tramite ``factory_class``),
il metodo sarà richiamato staticamente. Se il factory stesso va istanziato e il
metodo dell'oggetto risultante richiamato (come in questo esempio), configurare il
factory stesso come servizio:

.. configuration-block::

    .. code-block:: yaml

        parameters:
            # ...
            newsletter_manager.class: NewsletterManager
            newsletter_factory.class: NewsletterFactory
        services:
            newsletter_factory:
                class:            "%newsletter_factory.class%"
            newsletter_manager:
                class:            "%newsletter_manager.class%"
                factory_service:  newsletter_factory
                factory_method:   get

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

            <parameters>
                <!-- ... -->
                <parameter key="newsletter_manager.class">NewsletterManager</parameter>
                <parameter key="newsletter_factory.class">NewsletterFactory</parameter>
            </parameters>

            <services>
                <service id="newsletter_factory" class="%newsletter_factory.class%"/>

                <service
                    id="newsletter_manager"
                    class="%newsletter_manager.class%"
                    factory-service="newsletter_factory"
                    factory-method="get" />
            </services>
        </container>

    .. code-block:: php

        use Symfony\Component\DependencyInjection\Definition;

        // ...
        $container->setParameter('newsletter_manager.class', 'NewsletterManager');
        $container->setParameter('newsletter_factory.class', 'NewsletterFactory');

        $container->setDefinition('newsletter_factory', new Definition(
            '%newsletter_factory.class%'
        ));
        $container->setDefinition('newsletter_manager', new Definition(
            '%newsletter_manager.class%'
        ))->setFactoryService(
            'newsletter_factory'
        )->setFactoryMethod(
            'get'
        );

.. note::

   Il servizio factory è specificato dal suo nome id e non da un riferimento al servizio
   stesso. Non occorre quindi usare la sintassi con la chiocchiola nelle configurazioni
   YAML.

Passare parametri al metodo del factory
---------------------------------------

Se occorre passare parametri al metodo del factory, si può usare l'opzione ``arguments``
dentro al contenitore di servizi. Per esempio, si supponga che il metodo ``get``
dell'esempio precedente accetti un servizio ``templating`` come parametro:

.. configuration-block::

    .. code-block:: yaml

        parameters:
            # ...
            newsletter_manager.class: NewsletterManager
            newsletter_factory.class: NewsletterFactory
        services:
            newsletter_factory:
                class:            "%newsletter_factory.class%"
            newsletter_manager:
                class:            "%newsletter_manager.class%"
                factory_service:  newsletter_factory
                factory_method:   get
                arguments:
                    - "@templating"

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

            <parameters>
                <!-- ... -->
                <parameter key="newsletter_manager.class">NewsletterManager</parameter>
                <parameter key="newsletter_factory.class">NewsletterFactory</parameter>
            </parameters>

            <services>
                <service id="newsletter_factory" class="%newsletter_factory.class%"/>

                <service
                    id="newsletter_manager"
                    class="%newsletter_manager.class%"
                    factory-service="newsletter_factory"
                    factory-method="get">

                    <argument type="service" id="templating" />
                </service>
            </services>
        </container>

    .. code-block:: php

        use Symfony\Component\DependencyInjection\Definition;

        // ...
        $container->setParameter('newsletter_manager.class', 'NewsletterManager');
        $container->setParameter('newsletter_factory.class', 'NewsletterFactory');

        $container->setDefinition('newsletter_factory', new Definition(
            '%newsletter_factory.class%'
        ));
        $container->setDefinition('newsletter_manager', new Definition(
            '%newsletter_manager.class%',
            array(new Reference('templating'))
        ))->setFactoryService(
            'newsletter_factory'
        )->setFactoryMethod(
            'get'
        );
