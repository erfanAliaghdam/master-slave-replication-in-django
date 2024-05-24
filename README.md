# Setting Up Master-Slave Replication in Django

This guide will walk you through the process of setting up master-slave replication with Django.
Master-slave replication is a method of database replication where data from one database server
(the master) is automatically copied to one or more other database servers (the slaves).
This setup improves performance and provides fault tolerance by distributing the workload across multiple servers.

## Steps

### 1. Configure Django settings

In your Django project's settings file (`settings.py`), configure multiple database connections.
Define one connection for the master and one or more connections for the slaves.

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'master_db',
        'USER': 'master_user',
        'PASSWORD': 'master_password',
        'HOST': 'master_host',
        'PORT': 'master_port',
    },
    'slave': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'slave_db',
        'USER': 'slave_user',
        'PASSWORD': 'slave_password',
        'HOST': 'slave_host',
        'PORT': 'slave_port',
    },
    # Add more slave configurations if necessary
}
```

### 2. Define database routing

Create a database router to control which database each Django model uses for read and write operations.
This router will route write operations (e.g., `save()`, `delete()`) to the master database and read operations
(e.g., `objects.all()`, `objects.get()`) to the slave database(s).

```python
class MasterSlaveRouter:
    def db_for_read(self, model, **hints):
        """
        Attempts to read from the slave database(s).

        This method is called when Django needs to read data from the database.
        It returns the alias of the database to use for read operations.
        By default, it returns the 'slave' database alias.
        """
        return 'slave'

    def db_for_write(self, model, **hints):
        """
        Attempts to write to the master database.

        This method is called when Django needs to write data to the database.
        It returns the alias of the database to use for write operations.
        By default, it returns the 'default' database alias (master).
        """
        return 'default'

    def allow_relation(self, obj1, obj2, **hints):
        """
        Allows relations between objects of different databases.

        This method is called to determine if relations (foreign keys) between objects
        from different databases are allowed. By default, it returns None, indicating
        that relations between objects from different databases are not allowed.
        """
        return None

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        """
        Allows migrations for objects in the specified database.

        This method is called to determine if a migration operation (e.g., creating or
        deleting database tables) should be applied to the specified database. By default,
        it returns True, allowing migrations on all databases.
        """
        return True
```

### 3. Register the router

In your Django project's settings file (`settings.py`), register the custom database router.

```python
DATABASE_ROUTERS = ['path.to.your.MasterSlaveRouter']
```

### 4. Run migrations

After configuring the databases and router, run Django migrations to create the necessary database tables.

```bash
python manage.py migrate
```


