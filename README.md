# Instalação das Dependências

pip install fastapi sqlalchemy fastapi-pagination uvicorn


# Criação do Modelo de Dados

class Atleta(Base):
    __tablename__ = "atletas"

    id = Column(Integer, primary_key=True, autoincrement=True)
    nome = Column(String(50), nullable=False)
    cpf = Column(String(11), unique=True, nullable=False)
    centro_treinamento = Column(String(50))
    categoria = Column(String(300))

#Configuração do Banco de Dados:

DATABASE_URL = "postgresql://user:password@host:port/database"

engine = create_engine(DATABASE_URL)
Base.metadata.create_all(engine)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Endpoint para buscar todos os atletas

@app.get("/atletas", response_model=Page[Atleta])
async def get_all_atletas(db: Session = Depends(get_db), limit: int | None = None, offset: int | None = None):
    atletas = db.query(Atleta).order_by(Atleta.id).limit(limit).offset(offset).all()
    return Page(atletas, len(atletas))


# Endpoint para buscar atletas por nome e CPF

@app.get("/atletas/search", response_model=List[Atleta])
async def get_atleta_by_nome_cpf(nome: str | None = None, cpf: str | None = None, db: Session = Depends(get_db)):
    query = db.query(Atleta)

    if nome:
        query = query.filter(Atleta.nome.ilike(f"%{nome}%"))

    if cpf:
        query = query.filter(Atleta.cpf == cpf)

    atletas = query.all()

    if not atletas:
        raise HTTPException(status_code=404, detail="Atleta não encontrado")

    return atletas

