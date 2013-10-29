QDoctrine2 (Draft)
==================

QDoctrine2 is an object-relational mapper (ORM) for C++ (using Qt framework).

QDoctrine2 is a C++ port of the PHP library Doctrine2 (https://github.com/doctrine/doctrine2).

Configuration
=============

  1. Define connection parameters

          QHash<QString, QString> connectionParams;
          connectionParams["path"]   = "database.db";
          connectionParams["user"]   = "root";
          connectionParams["driver"] = "sqlite";

  2. Define entity annotations

          class User : public QObject
          {
              Q_OBJECT
    
              Q_CLASSINFO("User", "@Entity @Table(name='users')") // key = className
    
              Q_CLASSINFO("m_id", "@Id @Column(type='integer')")
              Q_CLASSINFO("m_id", "@GeneratedValue(strategy='AUTO')")
              Q_PROPERTY(int m_id READ getId WRITE setId)
    
              Q_CLASSINFO("m_name", "@Column(length=50)") // detault type : QString
              Q_PROPERTY(QString m_name READ getName WRITE setName)
              
              Q_CLASSINFO("m_projects", "@OneToMany(targetEntity='Project', mappedBy='m_user')") // OneToMany
              Q_PROPERTY(QList<Project*> m_projects READ getProjects WRITE setProjects)
          public:
              Q_INVOKABLE explicit User(QObject *parent = 0);
              ~User();
              
              int getId();
              const QString &getName();
              void setName(const QString &name);
              void setProjects(const QList<Project*> projects);
              const QList<Project*> &getProjects();
              void addProject(Project* project);
              void removeProject(Project* project);
              
          protected:
              int m_id;
              QString m_name;
              ToMany<Project> m_projects;
      
          private:
              void setId(int newId);
              
          };
      
          Q_DECLARE_METATYPE(User*)

  3. Initialize ORM

          qRegisterMetaType<User*>();
          qRegisterMetaType<Project*>();
      
          QStringList classes;
          classes << "User" << "Project";
          ORM::Configuration *config = new ORM::Configuration(&a);
          Common::Persistence::Mapping::Driver::MappingDriver *driver = config->newDefaultAnnotationDriver(classes);
          if(driver){
              config->setMetadataDriverImpl(driver);
              
              ORM::EntityManager *em = ORM::EntityManager::create(connectionParams, config);
              
              // work with entity manager...
          }

Usage
=====

  1. Get an entity

        User *u = em->find<User>("User", 1);
        if(u){
            qDebug() << "User : " << u->getName();
        }

  2. Update an entity

        User *u = em->find<User>("User", 1);
        if(u){
            u->setName("Another name.");
            em->persist(u);
        }

  3. Insert an entity

        User *newUser = new User();
        newUser->setName("new user");
        em->persist(newUser);
        em->flush();

  4. Remove an entity

        User *u = em->find<User>("User", 1);
        em->remove(u);
        em->flush();

  5. Use ManyToOne (By default: LAZY loading!)

        Project *project = em->find<Project>("Project", 1);
        if(project){
            qDebug() << "PROJECT :" << project->getName();
    
            if(project->getUser()){ // (the linked user entity is loaded here)
                qDebug() << "USER for the project : " << project->getUser()->getName();
            }else{
                qDebug() << "NO USER for the project";
            }
        }

  6. Use OneToMany (By default: LAZY loading!)

        User *u = em->find<User>("User", 1);
        if(u){
            QList<Project*> projects = u->getProjects(); // (the linked project entities are loaded here)
            for(int i = 0; i < projects.count(); i++)
            {
                qDebug() << projects[i]->getName();
            }
        }

TODO
====
- Finish implementation of collections (update/insert/remove)
- DQL implementation
- Repositories
- Tests
- ...
