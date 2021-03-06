
Alembic - Database Migration Tool
===================
This repository is a pre-configured extension of Alembic to facilitate hydrus DB Migrations. [Alembic](https://alembic.sqlalchemy.org/en/latest/) is a lightweight database migration tool for usage with the SQLAlchemy Database Toolkit for Python.

-----------------
First make sure you set up hydrus as the [instructions here](https://github.com/HTTP-APIs/hydrus/blob/master/README.md).

## Basic demo migration flow 

With the default hydrus clone run it with a sqlite database:

    hydrus serve --dburl sqlite:///hydrus/data/database.db

#### Point to your DB Url:

If following this tutorial, **the DB Url is already set accordingly SO SKIP THIS** . By default the DB URL points to your "DB_URL" Environment variable, or sqlite:///hydrus/data/database.db if no Env. Variable available. *If you want to change this* go to **hydrus/data/migrations/env.py** and edit the **DB_URL** variable so it points to your according DB Address.

	DB_URL = os.environ['DB_URL'] if 'DB_URL' in dict(os.environ).keys() else 'sqlite:///database.db'

#### Create migration script
Navigate to **...hydrus/hydrus/data** first, then run:

    alembic revision -m "creating obs. table and adding prop. column"
    
You should get something like this:

    Generating .../hydrus/hydrus/data/migrations/versions/f1359b4901c9__creating_obs_table_and_adding_prop_.py ... done

  That means it created a default migration script under the revision number of f1359b4901c9. Now it's time to write the wanted changes to the DB.

#### Adjust migration script  

In our example, we will be creating a new table and adding a column, head over to the new created file by the previous command under **.../hydrus/hydrus/data/migrations/versions/...** And add the following lines:
    
	def upgrade():
		op.create_table(
			'observations',
			sa.Column('id', sa.Integer, primary_key=True),
			sa.Column('drone_name', sa.String(50), nullable=False),
			sa.Column('observations', sa.Unicode(200)),
		)
		op.add_column('property', sa.Column('description', sa.String(20),
											server_default="Prop. Description"))


	def downgrade():
		op.drop_table('observations')
		op.drop_column('property', 'description')

**More information** on operations available to your migration here: https://alembic.sqlalchemy.org/en/latest/ops.html

#### Run migration script

To run the migration script run:

    alembic upgrade head

To go back with your migration run:

    alembic downgrade head

To go to a specific revision you can run, for example:

     alembic upgrade f1359b4901c9

*Obs.: If using SQLite you can just use https://sqliteonline.com/ to check the changes.*

## Auto Generating Migrations
Alembic can view the status of the database and compare against the table metadata in the application(hydrus.f), generating the “obvious” migrations based on a comparison. For that you have to provide the outdated SQL Database and also the sqlalchemy table metadata(hydrus.db_models.Base.metadata in hydrus). Both of them are already pre-configured in this repo, setting *target_metadata = hydrus.db_models.Base.metadata in env.py* and *sqlalchemy.url = sqlite:///database.db in alembic.ini*. **Change this to your preferences if necessary.**

### To create an automatic rudimentary migration script:
After changing the db schema at hydrus/data/db_models.py accordingly to your wish(and running *python setup.py install* again to update your build), run:

	alembic revision --autogenerate -m "autogenerated changes" 

You should get something *like hydrus/data/migrations/versions/0bca3dcf0826_autogenerated_changes.py* ... done. You should head over to this file and double-check/compliment your migration script. After this, just run your migration script. 

    alembic upgrade head


#### IMPORTANT - Quoting Alembic official page:

>> The vast majority of user issues with Alembic centers on the topic of what kinds of changes autogenerate can and cannot detect reliably, as well as how  it renders Python code for what it does detect. It is critical to note that autogenerate is not intended to be perfect. It is always necessary to manually review and correct the candidate migrations that autogenererate produces. The feature is getting more and more comprehensive and error-free as releases continue, but one should take note of the current limitations.

*To check exactly **what it can automatically do and cannot do**, please head over to: https://alembic.sqlalchemy.org/en/latest/autogenerate.html*

##  Full-featured migrations, troubleshoot and updates

There's always an updated full getting start tutorial on Alembic here: https://alembic.sqlalchemy.org/en/latest/tutorial.html
 *Also head over to the official Alembic page for all the possibilities you have:* https://alembic.sqlalchemy.org/

