# graphql-meetup

Un petit repo en assistant pour le meetup GraphQL

## Les exemples de query sur la swapi

La super simple :
	
	{
	  person(personID:1){
	    name
	  }
	}

La complète Jambon/Fromage/Oeuf :

	{
	  person(personID:1){
	    name,
	    height,
	    mass,
	    skinColor,
	    eyeColor,
	    birthYear,
	    gender,
	    created,
	    edited,
	    homeworld {
	      id,
	      name
	    },
	    filmConnection{
	      films{
	        id,
	        title
	      }
	    },
	    species{
	      id,
	      name
	    },
	    vehicleConnection{
	      vehicles{
	        id,
	        name
	      }
	    },
	    starshipConnection{
	      starships{
	        id,
	        name
	      }
	    }
	  }
	}

## La partie prisma

### Prérequis

docker-compose et npm dans sa dernière version. Puis on installe prisma :

	npm i -g prisma

### Les manipulation

On initialise le projet 

	prisma init manga
	
On choisi de créer un projet basé sur docker pour notre exemple. Il faut donc lancer le docker. Pour ça, dans le repertoire qui vient de se créer :

	docker-compose up -d

On modifie datamodel.prisma

	type Manga {
	  id: ID! @unique
	  title: String! @unique
	  category: Category!
	  author: Author!
	  volumes: [Volume!]!
	}
	
	type Author {
	  id: ID! @unique
	  name: String! @unique
	  bio: String
	  mangas: [Manga!]!
	}
	
	type Volume {
	  id: ID! @unique
	  title: String
	  number: Int! @unique
	  manga: Manga!
	}
	
	type Category {
	  id: ID! @unique
	  name: String! @unique
	  mangas: [Manga!]!
	}

On déploie cette modification :

	prisma deploy
	
On profite du graphql sur http://localhost:4466

### La création de données

# Write your query or mutation here

	mutation createChefOeuvre{createCategory(data:{name:"chef d'oeuvre"}){id, name}}
	mutation createHumour{createCategory(data:{name:"humour"}){id, name}}
	
	mutation createAkira {
	  createAuthor(data:{
	    name:"Akira TORIYAMA", 
	    bio:"Akira Toriyama (鳥山 明, Toriyama Akira) est un auteur de manga et character designer japonais né le 5 avril 1955 à Nagoya dans la préfecture d'Aichi."
	    mangas:{
	      create:[{
	        title:"Dragon Ball", 
	        category:{connect:{name:"chef d'oeuvre"}},
	        volumes:{create:
	          [
	            {title:"Volume 1 - Dragon Ball", number: 1},
	            {title:"Volume 2 - Dragon Ball", number: 2}
	          ]
	        }
	      },
	      {
	        title:"Dr SLUMP", 
	        category:{connect:{name:"humour"}},
	        volumes:{create:
	          [{title:"Volume 1 - Dr SLUMP", number: 1},{title:"Volume 2 - Dr SLUMP", number: 2}]
	        }
	      }
	      ]
	    }
	  }
	  ){
	    id
	  }
	}
	
	mutation createOda {
	  createAuthor(data:{
	    name:"Eiichirō Oda",
	        mangas:{
	      create:[{
	        title:"One PIECE", 
	        category:{connect:{name:"chef d'oeuvre"}},
	        volumes:{create:
	          [{title:"Volume 1 - One Piece", number: 1},{title:"Volume 2 - One Piece", number: 2}]
	        }
	      }
	      ]
	    }
	  }
	  ){
	    id
	  }
	}

### Les requètes de démo

	query listMangas{mangas{id,title}}
	
	query listAuthors{authors{id, name, bio, mangas{title}}}
	
	query toutLesMangasDroles{
	  category(where:{name:"humour"}){
	    mangas{title}
	  }
	}
	
	query complete{
	  author(where:{name:"Eiichirō Oda"}){
	    id,
	    name,
	    mangas{
	      id, 
	      title,
	      category{
	        id,
	        mangas{
	          title,author{name}
	        }
	      }
	    }
	  }
	}

