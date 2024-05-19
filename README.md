# -Desafio-de-Projeto-Desenvolvendo-Primeira-API-com-FastAPI-Python-e-Docker-

Postagens das partes que compõem o projeto:
A) Atleta;
rom fastapi import FastAPI

app = FastAPI(title='workaoutapi')

if name"main":
import uvicorn
uvicorn.run(app, host= '0, 0, 0, port=8000')
from typing import Annotated, Optional
from pydantic import Field, PositiveFloat
from workout_api.categorias.schemas import CategoriaIn
from workout_api.centro_treinamento.schemas import CentroTreinamentoAtleta

from workout_api.contrib.schemas import BaseSchema, OutMixin

class Atleta(BaseSchema):
nome: Annotated[str, Field(description='Nome do atleta', example='Joao', max_length=50)]
cpf: Annotated[str, Field(description='CPF do atleta', example='12345678900', max_length=11)]
idade: Annotated[int, Field(description='Idade do atleta', example=25)]
peso: Annotated[PositiveFloat, Field(description='Peso do atleta', example=75.5)]
altura: Annotated[PositiveFloat, Field(description='Altura do atleta', example=1.70)]
sexo: Annotated[str, Field(description='Sexo do atleta', example='M', max_length=1)]
categoria: Annotated[CategoriaIn, Field(description='Categoria do atleta')]
centro_treinamento: Annotated[CentroTreinamentoAtleta, Field(description='Centro de treinamento do atleta')]
class AtletaIn(Atleta):
pass

class AtletaOut(Atleta, OutMixin):
pass

class AtletaUpdate(BaseSchema):
nome: Annotated[Optional[str], Field(None, description='Nome do atleta', example='Joao', max_length=50)]
idade: Annotated[Optional[int], Field(None, description='Idade do atleta', example=25)]

class AtletaModel(BaseModel):
tablename = 'atletas'

pk_id: Mapped[int] = mapped_column(Integer, primary_key=True)
nome: Mapped[str] = mapped_column(String(50), nullable=False)
cpf: Mapped[str] = mapped_column(String(11), unique=True, nullable=False)
idade: Mapped[int] = mapped_column(Integer, nullable=False)
peso: Mapped[float] = mapped_column(Float, nullable=False)
altura: Mapped[float] = mapped_column(Float, nullable=False)
sexo: Mapped[str] = mapped_column(String(1), nullable=False)
created_at: Mapped[datetime] = mapped_column(DateTime, nullable=False)
categoria: Mapped['CategoriaModel'] = relationship(back_populates="atleta", lazy='selectin')
categoria_id: Mapped[int] = mapped_column(ForeignKey("categorias.pk_id"))
centro_treinamento: Mapped['CentroTreinamentoModel'] = relationship(back_populates="atleta", lazy='selectin')
centro_treinamento_id: Mapped[int] = mapped_column(ForeignKey("centros_treinamento.pk_id"))

class Atleta(BaseSchema):
nome: Annotated[str, Field(description='Nome do atleta', example='Joao', max_length=50)]
cpf: Annotated[str, Field(description='CPF do atleta', example='12345678900', max_length=11)]
idade: Annotated[int, Field(description='Idade do atleta', example=25)]
peso: Annotated[PositiveFloat, Field(description='Peso do atleta', example=75.5)]
altura: Annotated[PositiveFloat, Field(description='Altura do atleta', example=1.70)]
sexo: Annotated[str, Field(description='Sexo do atleta', example='M', max_length=1)]
categoria: Annotated[CategoriaIn, Field(description='Categoria do atleta')]
centro_treinamento: Annotated[CentroTreinamentoAtleta, Field(description='Centro de treinamento do atleta')]

class AtletaIn(Atleta):
pass

class AtletaOut(Atleta, OutMixin):
pass
class AtletaUpdate(BaseSchema):
nome: Annotated[Optional[str], Field(None, description='Nome do atleta', example='Joao', max_length=50)]
idade: Annotated[Optional[int], Field(None, description='Idade do atleta', example=25)]

version: "3"
services:
db:
image: postgres:11-alpine
environment:
POSTGRES_PASSWORD: workout
POSTGRES_USER: workout
POSTGRES_DB: workout
ports:
- "5432:5432"
@router.post(
'/',
summary='Criar um novo atleta',
status_code=status.HTTP_201_CREATED,
response_model=AtletaOut
)
async def post(
db_session: DatabaseDependency,
atleta_in: AtletaIn = Body(...)
):
categoria_nome = atleta_in.categoria.nome
centro_treinamento_nome = atleta_in.centro_treinamento.nome
categoria = (await db_session.execute(
select(CategoriaModel).filter_by(nome=categoria_nome))
).scalars().first()

if not categoria:
raise HTTPException(
status_code=status.HTTP_400_BAD_REQUEST,
detail=f'A categoria {categoria_nome} não foi encontrada.'
)

centro_treinamento = (await db_session.execute(
select(CentroTreinamentoModel).filter_by(nome=centro_treinamento_nome))
).scalars().first()

if not centro_treinamento:
raise HTTPException(
status_code=status.HTTP_400_BAD_REQUEST,
detail=f'O centro de treinamento {centro_treinamento_nome} não foi encontrado.'
)
try:
atleta_out = AtletaOut(id=uuid4(), created_at=datetime.utcnow(), **atleta_in.model_dump())
atleta_model = AtletaModel(**atleta_out.model_dump(exclude={'categoria', 'centro_treinamento'}))

  atleta_model.categoria_id = categoria.pk_id
  atleta_model.centro_treinamento_id = centro_treinamento.pk_id

  db_session.add(atleta_model)
  await db_session.commit()

  except Exception:
raise HTTPException(
status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
detail='Ocorreu um erro ao inserir os dados no banco'
)

return atleta_out

@router.get(
'/',
summary='Consultar todos os Atletas',
status_code=status.HTTP_200_OK,
response_model=list[AtletaOut],
)
async def query(db_session: DatabaseDependency) -> list[AtletaOut]:
atletas: list[AtletaOut] = (await db_session.execute(select(AtletaModel))).scalars().all()

return [AtletaOut.model_validate(atleta) for atleta in atletas]

@router.get(
'/{id}',
summary='Consulta um Atleta pelo id',
status_code=status.HTTP_200_OK,
response_model=AtletaOut,
)
async def get(id: UUID4, db_session: DatabaseDependency) -> AtletaOut:
atleta: AtletaOut = (
await db_session.execute(select(AtletaModel).filter_by(id=id))
).scalars().first()

if not atleta:
raise HTTPException(
status_code=status.HTTP_404_NOT_FOUND,
detail=f'Atleta não encontrado no id: {id}'
)

return atleta

@router.patch(
'/{id}',
summary='Editar um Atleta pelo id',
status_code=status.HTTP_200_OK,
response_model=AtletaOut,
)
async def patch(id: UUID4, db_session: DatabaseDependency, atleta_up: AtletaUpdate = Body(...)) -> AtletaOut:
atleta: AtletaOut = (
await db_session.execute(select(AtletaModel).filter_by(id=id))
).scalars().first()

if not atleta:
raise HTTPException(
status_code=status.HTTP_404_NOT_FOUND,
detail=f'Atleta não encontrado no id: {id}'
)

atleta_update = atleta_up.model_dump(exclude_unset=True)
for key, value in atleta_update.items():
setattr(atleta, key, value)

await db_session.commit()
await db_session.refresh(atleta)

return atleta

@router.delete(
'/{id}',
summary='Deletar um Atleta pelo id',
status_code=status.HTTP_204_NO_CONTENT
)
async def delete(id: UUID4, db_session: DatabaseDependency) -> None:
atleta: AtletaOut = (
await db_session.execute(select(AtletaModel).filter_by(id=id))
).scalars().first()

if not atleta:
raise HTTPException(
status_code=status.HTTP_404_NOT_FOUND,
detail=f'Atleta não encontrado no id: {id}'
)

await db_session.delete(atleta)
await db_session.commit()

B) Categorias;

class CategoriaModel(BaseModel):
tablename = 'categorias'

pk_id: Mapped[int] = mapped_column(Integer, primary_key=True)
nome: Mapped[str] = mapped_column(String(50), unique=True, nullable=False)
atleta: Mapped['AtletaModel'] = relationship(back_populates="categoria")

from typing import Annotated

from pydantic import UUID4, Field
from workout_api.contrib.schemas import BaseSchema

class CategoriaIn(BaseSchema):
nome: Annotated[str, Field(description='Nome da categoria', example='Scale', max_length=10)]

class CategoriaOut(CategoriaIn):
id: Annotated[UUID4, Field(description='Identificador da categoria')]

@router.post(
'/',
summary='Criar uma nova Categoria',
status_code=status.HTTP_201_CREATED,
response_model=CategoriaOut,
)
async def post(
db_session: DatabaseDependency,
categoria_in: CategoriaIn = Body(...)
) -> CategoriaOut:
categoria_out = CategoriaOut(id=uuid4(), **categoria_in.model_dump())
categoria_model = CategoriaModel(**categoria_out.model_dump())

db_session.add(categoria_model)
await db_session.commit()

return categoria_out

@router.get(
'/',
summary='Consultar todas as Categorias',
status_code=status.HTTP_200_OK,
response_model=list[CategoriaOut],
)
async def query(db_session: DatabaseDependency) -> list[CategoriaOut]:
categorias: list[CategoriaOut] = (await db_session.execute(select(CategoriaModel))).scalars().all()

return categorias

@router.get(
'/{id}',
summary='Consulta uma Categoria pelo id',
status_code=status.HTTP_200_OK,
response_model=CategoriaOut,
)
async def get(id: UUID4, db_session: DatabaseDependency) -> CategoriaOut:
categoria: CategoriaOut = (
await db_session.execute(select(CategoriaModel).filter_by(id=id))
).scalars().first()

if not categoria:
raise HTTPException(
status_code=status.HTTP_404_NOT_FOUND,
detail=f'Categoria não encontrada no id: {id}'
)

return categoria

C) Centro de Treinamento;

from sqlalchemy import Integer, String
from sqlalchemy.orm import Mapped, mapped_column, relationship
from workout_api.contrib.models import BaseModel
from workout_api.atleta.models import AtletaModel

class CentroTreinamentoModel(BaseModel):
tablename = 'centros_treinamento'

pk_id: Mapped[int] = mapped_column(Integer, primary_key=True)
nome: Mapped[str] = mapped_column(String(50), unique=True, nullable=False)
endereco: Mapped[str] = mapped_column(String(60), nullable=False)
proprietario: Mapped[str] = mapped_column(String(30), nullable=False)
atleta: Mapped['AtletaModel'] = relationship(back_populates='centro_treinamento')

ss CentroTreinamentoModel(BaseModel):
tablename = 'centros_treinamento'

pk_id: Mapped[int] = mapped_column(Integer, primary_key=True)
nome: Mapped[str] = mapped_column(String(50), unique=True, nullable=False)
endereco: Mapped[str] = mapped_column(String(60), nullable=False)
proprietario: Mapped[str] = mapped_column(String(30), nullable=False)
atleta: Mapped['AtletaModel'] = relationship(back_populates='centro_treinamento')

m typing import Annotated

from pydantic import Field, UUID4
from workout_api.contrib.schemas import BaseSchema

class CentroTreinamentoIn(BaseSchema):
nome: Annotated[str, Field(description='Nome do centro de treinamento', example='CT King', max_length=20)]
endereco: Annotated[str, Field(description='Endereço do centro de treinamento', example='Rua X, Q02', max_length=60)]
proprietario: Annotated[str, Field(description='Proprietario do centro de treinamento', example='Marcos', max_length=30)]

class CentroTreinamentoAtleta(BaseSchema):
nome: Annotated[str, Field(description='Nome do centro de treinamento', example='CT King', max_length=20)]

class CentroTreinamentoOut(CentroTreinamentoIn):
id: Annotated[UUID4, Field(description='Identificador do centro de treinamento')]

from uuid import uuid4
from fastapi import APIRouter, Body, HTTPException, status
from pydantic import UUID4
from workout_api.centro_treinamento.schemas import CentroTreinamentoIn, CentroTreinamentoOut
from workout_api.centro_treinamento.models import CentroTreinamentoModel

from workout_api.contrib.dependencies import DatabaseDependency
from sqlalchemy.future import select

router = APIRouter()

@router.post(
'/',
summary='Criar um novo Centro de treinamento',
status_code=status.HTTP_201_CREATED,
response_model=CentroTreinamentoOut,
)
async def post(
db_session: DatabaseDependency,
centro_treinamento_in: CentroTreinamentoIn = Body(...)
) -> CentroTreinamentoOut:
centro_treinamento_out = CentroTreinamentoOut(id=uuid4(), **centro_treinamento_in.model_dump())
centro_treinamento_model = CentroTreinamentoModel(**centro_treinamento_out.model_dump())

db_session.add(centro_treinamento_model)
await db_session.commit()

return centro_treinamento_out

@router.get(
'/',
summary='Consultar todos os centros de treinamento',
status_code=status.HTTP_200_OK,
response_model=list[CentroTreinamentoOut],
)
async def query(db_session: DatabaseDependency) -> list[CentroTreinamentoOut]:
centros_treinamento_out: list[CentroTreinamentoOut] = (
await db_session.execute(select(CentroTreinamentoModel))
).scalars().all()

return centros_treinamento_out

@router.get(
'/{id}',
summary='Consulta um centro de treinamento pelo id',
status_code=status.HTTP_200_OK,
response_model=CentroTreinamentoOut,
)
async def get(id: UUID4, db_session: DatabaseDependency) -> CentroTreinamentoOut:
centro_treinamento_out: CentroTreinamentoOut = (
await db_session.execute(select(CentroTreinamentoModel).filter_by(id=id))
).scalars().first()

if not centro_treinamento_out:
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND, 
        detail=f'Centro de treinamento não encontrado no id: {id}'
    )

return centro_treinamento_out

D) Configurações

from typing import AsyncGenerator

from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker
from workout_api.configs.settings import settings

engine = create_async_engine(settings.DB_URL, echo=False)
async_session = sessionmaker(
engine, class_=AsyncSession, expire_on_commit=False
)

async def get_session() -> AsyncGenerator:
async with async_session() as session:
yield session
from pydantic import Field
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
DB_URL: str = Field(default='postgresql+asyncpg://workout:workout@localhost/workout')

settings = Settings()

