## FastAPI 最佳实践
我们在初创公司使用的最佳实践和惯例的意见列表。

在过去 1.5 年的生产环境中，我们一直在做出好的和坏的决定，这些决定极大地影响了我们的开发人员体验。其中一些值得分享。

### 内容
1. [项目结构。一致且可预测。](https://github.com/zhanymkanov/fastapi-best-practices#1-project-structure-consistent--predictable)
2. [更多地使用Pydantic进行数据验证](https://github.com/zhanymkanov/fastapi-best-practices#2-excessively-use-pydantic-for-data-validation)
3. [使用依赖项来进行数据库相关的数据验证](https://github.com/zhanymkanov/fastapi-best-practices#3-use-dependencies-for-data-validation-vs-db)
4. [依赖链](https://github.com/zhanymkanov/fastapi-best-practices#4-chain-dependencies)
5. [解耦和重用依赖关系。依赖项调用将会被缓存。](https://github.com/zhanymkanov/fastapi-best-practices#5-decouple--reuse-dependencies-dependency-calls-are-cached)
6. [遵循 REST 规范](https://github.com/zhanymkanov/fastapi-best-practices#6-follow-the-rest)
7. [如果你的路由处理函数中含有阻塞 I/O 操作，不要让你的路由异步.](https://github.com/zhanymkanov/fastapi-best-practices#7-dont-make-your-routes-async-if-you-have-only-blocking-io-operations)
8. [定制自己的基础模型](https://github.com/zhanymkanov/fastapi-best-practices#8-custom-base-model-from-day-0)
9. [文档.](https://github.com/zhanymkanov/fastapi-best-practices#9-docs)
10. [10. 使用 Pydantic 的 BaseSettings 进行配置](https://github.com/zhanymkanov/fastapi-best-practices#10-use-pydantics-basesettings-for-configs)
11. [SQLAlchemy：设置数据库键命名约定](https://github.com/zhanymkanov/fastapi-best-practices#11-sqlalchemy-set-db-keys-naming-convention)
12. [Migrations. Alembic.](https://github.com/zhanymkanov/fastapi-best-practices#12-migrations-alembic)
13. [设置数据库命名约定](https://github.com/zhanymkanov/fastapi-best-practices#13-set-db-naming-convention)
14. [设置异步测试客户端.](https://github.com/zhanymkanov/fastapi-best-practices#14-set-tests-client-async-from-day-0)
15. [后台任务 > asyncio.create_task.](https://github.com/zhanymkanov/fastapi-best-practices#15-backgroundtasks--asynciocreate_task)
16. [Typing 非常重要.](https://github.com/zhanymkanov/fastapi-best-practices#16-typing-is-important)
17. [以块的形式保存文件.](https://github.com/zhanymkanov/fastapi-best-practices#17-save-files-in-chunks)
18. [ 小心动态 pydantic 字段 (Pydantic v1)](https://github.com/zhanymkanov/fastapi-best-practices#18-be-careful-with-dynamic-pydantic-fields)
19. [SQL优先，Pydantic其次](https://github.com/zhanymkanov/fastapi-best-practices#19-sql-first-pydantic-second) 
20. [验证主机（如果用户可以发送公开可用的 URL）](https://github.com/zhanymkanov/fastapi-best-practices#20-validate-hosts-if-users-can-send-publicly-available-urls)
21. [在自定义pydantic 验证器中直接触发 ValueError](https://github.com/zhanymkanov/fastapi-best-practices#21-raise-a-valueerror-in-custom-pydantic-validators-if-schema-directly-faces-the-client)
22. [FastAPI将Pydantic对象转换为dict，然后转换为Pydantic对象，然后转换为JSON](https://github.com/zhanymkanov/fastapi-best-practices#22-fastapi-converts-pydantic-objects-to-dict-then-to-pydantic-object-then-to-json)
23. [如果必须使用sync SDK，那么在线程池中运行它](https://github.com/zhanymkanov/fastapi-best-practices#23-if-you-must-use-sync-sdk-then-run-it-in-a-thread-pool)
24. [Use linters (black, ruff).](https://github.com/zhanymkanov/fastapi-best-practices#24-use-linters-black-ruff)
25. [Bonus Section.](https://github.com/zhanymkanov/fastapi-best-practices#bonus-section)
<p style="text-align: center;"> <i>Project <a href="https://github.com/zhanymkanov/fastapi_production_template">sample</a> built with these best-practices in mind. </i> </p>

### 1.  项目结构，一致且可预测
构建项目的方法有很多种，但最好的结构是一致、简单且没有出乎意料的结构。

- 如果查看项目结构无法让您了解项目的内容，那么结构是不清晰的。 
- 如果您必须打开包才能了解其中包含哪些模块，那么您的结构就不清晰的。
- 如果文件的频率和位置感觉是随机的，那么您的项目结构就很糟糕。
- 如果查看模块的位置和名称并不能让您了解其中的内容，那么您的结构就非常糟糕。

虽然 [@tiangolo](https://github.com/tiangolo) 提出的项目结构（我们按类型（例如 api、crud、模型、模式）分隔文件）对于微服务或范围较小的项目很有用，但我们无法将其放入具有大量的域和模块。我受到 Netflix 的[Dispatch](https://github.com/Netflix/dispatch)的启发，发现了和种更具可扩展性和可进化性的结构并做了一些小修改。

```
fastapi-project
├── alembic/
├── src
│   ├── auth
│   │   ├── router.py
│   │   ├── schemas.py  # pydantic 模型
│   │   ├── models.py  # db 模型
│   │   ├── dependencies.py
│   │   ├── config.py  # 本地配置
│   │   ├── constants.py
│   │   ├── exceptions.py
│   │   ├── service.py
│   │   └── utils.py
│   ├── aws
│   │   ├── client.py  # client model for external service communication
│   │   ├── schemas.py
│   │   ├── config.py
│   │   ├── constants.py
│   │   ├── exceptions.py
│   │   └── utils.py
│   └── posts
│   │   ├── router.py
│   │   ├── schemas.py
│   │   ├── models.py
│   │   ├── dependencies.py
│   │   ├── constants.py
│   │   ├── exceptions.py
│   │   ├── service.py
│   │   └── utils.py
│   ├── config.py  # 全局配置
│   ├── models.py  # 全局模型
│   ├── exceptions.py  # 全局异常
│   ├── pagination.py  # global module e.g. pagination
│   ├── database.py  # 数据库连接相关
│   └── main.py
├── tests/
│   ├── auth
│   ├── aws
│   └── posts
├── templates/
│   └── index.html
├── requirements
│   ├── base.txt
│   ├── dev.txt
│   └── prod.txt
├── .env
├── .gitignore
├── logging.ini
└── alembic.ini
```
1. 将所有域目录存储在 `src` 文件夹 内
   1. `src/` - 应用程序的最高层目录，包含常见模型、配置和常量等。
   2. `src/main.py` - 项目的根目录，用于初始化 FastAPI 应用程序
2. 每个包都有自己的路由器、模式、模型等
   1. `router.py` - 是每个模块的核心，具有所有请求接入点
   2. `schemas.py` - 适用于 pydantic 模型
   3. `models.py` - 数据库模型
   4. `service.py` - 模块特定的业务逻辑  
   5. `dependencies.py` - 路由的依赖项
   6. `constants.py` - 模块级别的特定常量和错误代码
   7. `config.py` - 模块级别的配置项，例如环境变量
   8. `utils.py` - 非业务逻辑功能，例如响应规范化、数据丰富等。
   9. `exceptions.py` -  定义模块级别特定的异常，例如 `PostNotFound`, `InvalidUserData`
3. 当某个包中需要来自其他包的服务或依赖项或常量时 - 使用显式模块名称导入它们
```python
from src.auth import constants as auth_constants
from src.notifications import service as notification_service
from src.posts.constants import ErrorCode as PostsErrorCode  # in case we have Standard ErrorCode in constants module of each package
```

### 2. 更多地使用Pydantic进行数据验证
Pydantic 具有丰富的功能来验证和转换数据。

除了一些常见的校验内容，如必填项和有默认值的非必填字段等常规功能之外，Pydantic 还内置了丰富全面的数据处理工具，例如正则表达式、限制允许选项的枚举、长度验证、电​​子邮件验证等。

```python3
from enum import Enum
from pydantic import AnyUrl, BaseModel, EmailStr, Field, constr

# 只允许"AEROSMITH", "QUEEN", "AC/DC" 这三个选项
class MusicBand(str, Enum):
   AEROSMITH = "AEROSMITH"
   QUEEN = "QUEEN"
   ACDC = "AC/DC"


class UserBase(BaseModel):
    first_name: str = Field(min_length=1, max_length=128)
    username: constr(regex="^[A-Za-z0-9-_]+$", to_lower=True, strip_whitespace=True)
    email: EmailStr
    age: int = Field(ge=18, default=None)  # must be greater or equal to 18
    favorite_band: MusicBand = None  # only "AEROSMITH", "QUEEN", "AC/DC" values are allowed to be inputted
    website: AnyUrl = None

```
### 3.使用依赖项来进行数据库相关的数据验证
Pydantic 只能校验客户端输入的值的有效性。如长度大小等，当用户的输入是合法的，但是用户的输入需要在数据库中进行查询，如获取某篇文章，但是这篇文章可能在数据库中是不存的，这时我们需要使用依赖项进行数据库的约束（例如电子邮件已存在、用户未找到等）验证数据。

```python3
# dependencies.py
async def valid_post_id(post_id: UUID4) -> Mapping:
    post = await service.get_by_id(post_id)
    if not post:
        raise PostNotFound()

    return post


# router.py
@router.get("/posts/{post_id}", response_model=PostResponse)
async def get_post_by_id(post: Mapping = Depends(valid_post_id)):
    return post


@router.put("/posts/{post_id}", response_model=PostResponse)
async def update_post(
    update_data: PostUpdate,  
    post: Mapping = Depends(valid_post_id), 
):
    updated_post: Mapping = await service.update(id=post["id"], data=update_data)
    return updated_post


@router.get("/posts/{post_id}/reviews", response_model=list[ReviewsResponse])
async def get_post_reviews(post: Mapping = Depends(valid_post_id)):
    post_reviews: list[Mapping] = await reviews_service.get_by_post_id(post["id"])
    return post_reviews
```
如果我们不将数据验证置于依赖项中，则必须为每个接口请求添加 post_id 验证，并为每个接口处理请求编写相同的测试。

### 4. 依赖链
依赖荐可以使用其他依赖荐，这样可以避免类似重复的逻辑代码。

```python3
# dependencies.py
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt

async def valid_post_id(post_id: UUID4) -> Mapping:
    post = await service.get_by_id(post_id)
    if not post:
        raise PostNotFound()

    return post


async def parse_jwt_data(
    token: str = Depends(OAuth2PasswordBearer(tokenUrl="/auth/token"))
) -> dict:
    try:
        payload = jwt.decode(token, "JWT_SECRET", algorithms=["HS256"])
    except JWTError:
        raise InvalidCredentials()

    return {"user_id": payload["id"]}


async def valid_owned_post(
    post: Mapping = Depends(valid_post_id), 
    token_data: dict = Depends(parse_jwt_data),
) -> Mapping:
    if post["creator_id"] != token_data["user_id"]:
        raise UserNotOwner()

    return post

# router.py
@router.get("/users/{user_id}/posts/{post_id}", response_model=PostResponse)
async def get_user_post(post: Mapping = Depends(valid_owned_post)):
    return post

```
### 5. 解耦和重用依赖关系。依赖项调用将会被缓存。
依赖项可以被多次使用，并且不会重新计算 - FastAPI 默认情况下会在请求范围内缓存依赖项的结果，即如果我们有一个调用 service 的依赖项 `get_post_by_id`，则每次调用此依赖项时我们都不会访问数据库 - 仅在第一次函数调用时才会访问数据库。

知道了这一点，我们可以轻松地将依赖关系解耦到多个较小的函数，这些函数在较小的域上运行，并且更容易在其他路由中重用。例如，在下面的代码中我们使用了parse_jwt_data三次：

1. `valid_owned_post`
2. `valid_active_creator`
3. `get_user_post`,

but `parse_jwt_data` is called only once, in the very first call.

```python3
# dependencies.py
from fastapi import BackgroundTasks
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt

async def valid_post_id(post_id: UUID4) -> Mapping:
    post = await service.get_by_id(post_id)
    if not post:
        raise PostNotFound()

    return post


async def parse_jwt_data(
    token: str = Depends(OAuth2PasswordBearer(tokenUrl="/auth/token"))
) -> dict:
    try:
        payload = jwt.decode(token, "JWT_SECRET", algorithms=["HS256"])
    except JWTError:
        raise InvalidCredentials()

    return {"user_id": payload["id"]}


async def valid_owned_post(
    post: Mapping = Depends(valid_post_id), 
    token_data: dict = Depends(parse_jwt_data),
) -> Mapping:
    if post["creator_id"] != token_data["user_id"]:
        raise UserNotOwner()

    return post


async def valid_active_creator(
    token_data: dict = Depends(parse_jwt_data),
):
    user = await users_service.get_by_id(token_data["user_id"])
    if not user["is_active"]:
        raise UserIsBanned()
    
    if not user["is_creator"]:
       raise UserNotCreator()
    
    return user
        

# router.py
@router.get("/users/{user_id}/posts/{post_id}", response_model=PostResponse)
async def get_user_post(
    worker: BackgroundTasks,
    post: Mapping = Depends(valid_owned_post),
    user: Mapping = Depends(valid_active_creator),
):
    """Get post that belong the active user."""
    worker.add_task(notifications_service.send_email, user["id"])
    return post

```

### 6. 遵循 REST 规范
开发 RESTful API 可以更轻松地重用路由中的依赖项，如下所示:
   1. `GET /courses/:course_id`
   2. `GET /courses/:course_id/chapters/:chapter_id/lessons`
   3. `GET /chapters/:chapter_id`

唯一需要注意的是在路径中使用相同的变量名称:
- 如果您有两个接口路由，`GET /profiles/:profile_id`，`GET /creators/:creator_id`, 并且 `GET /creators/:creator_id` 既验证给定的参数profile_id 是否存在，又检查配置文件是否是创建者，那么最好将creator_id路径变量重命名为profile_id并链接这两个依赖项。

```python3
# src.profiles.dependencies
async def valid_profile_id(profile_id: UUID4) -> Mapping:
    profile = await service.get_by_id(profile_id)
    if not profile:
        raise ProfileNotFound()

    return profile

# src.creators.dependencies
async def valid_creator_id(profile: Mapping = Depends(valid_profile_id)) -> Mapping:
    if not profile["is_creator"]:
       raise ProfileNotCreator()

    return profile

# src.profiles.router.py
@router.get("/profiles/{profile_id}", response_model=ProfileResponse)
async def get_user_profile_by_id(profile: Mapping = Depends(valid_profile_id)):
    """Get profile by id."""
    return profile

# src.creators.router.py
@router.get("/creators/{profile_id}", response_model=ProfileResponse)
async def get_user_profile_by_id(
     creator_profile: Mapping = Depends(valid_creator_id)
):
    """Get creator's profile by id."""
    return creator_profile

```

对用户资源使用 /me 端点（例如`GET /profiles/me`, `GET /users/me/posts`）
   1. 无需验证用户 ID 是否存在 - 已通过 auth 方法检查过
   2. 无需检查用户id是否属于请求者

### 7. 如果你的路由处理函数中含有阻塞 I/O 操作，不要让你的路由异步


- FastAPI runs `sync` routes in the [threadpool](https://en.wikipedia.org/wiki/Thread_pool) 
and blocking I/O operations won't stop the [event loop](https://docs.python.org/3/library/asyncio-eventloop.html) 
from executing the tasks. 
- Otherwise, if the route is defined `async` then it's called regularly via `await` 
and FastAPI trusts you to do only non-blocking I/O operations.

The caveat is if you fail that trust and execute blocking operations within async routes, 
the event loop will not be able to run the next tasks until that blocking operation is done.

在底层，FastAPI 可以[有效地处理](https://fastapi.tiangolo.com/async/#path-operation-functions)异步和同步 I/O 操作。

- FastAPI 在[线程池](https://en.wikipedia.org/wiki/Thread_pool)中运行`sync` 路由 ，阻塞 I/O 操作不会阻止[事件循环](https://docs.python.org/3/library/asyncio-eventloop.html) 执行任务。
- 否则，如果使用`async` 定义了路由处理函数，并且通过`await` 来调用，FastAPI 会认为您只执行非阻塞 I/O 操作。
需要注意的是，如果在异步路由中执行阻塞操作，则事件循环将无法运行下一个任务，直到该阻塞操作完成为止。

```python
import asyncio
import time

@router.get("/terrible-ping")
async def terrible_catastrophic_ping(): # 这个是使用 async定义的
    time.sleep(10) # I/O 阻塞 10 秒钟
    pong = service.get_pong()  # 从DB中获取数据的 I/O 阻塞操作，
    
    return {"pong": pong}

@router.get("/good-ping")
def good_ping():  # 这个是没有使用 async 定义的
    time.sleep(10) # I/O 阻塞 10 秒钟，但是它是在另外的线程中执行
    pong = service.get_pong()  # 从DB中获取数据的 I/O 阻塞操作，但是它是在另外的线程中执行，上面两个操作均不会阻塞整个系统
    
    return {"pong": pong}

@router.get("/perfect-ping")
async def perfect_ping():
    await asyncio.sleep(10) # 非阻塞 I/O 操作
    pong = await service.async_get_pong()  # 非阻塞的DB I/O 操作

    return {"pong": pong}

```
**当我们调用时会发生什么:**
1. `GET /terrible-ping` (async def 定义的函数中含有阻塞IO操作)
   1. FastAPI 服务器接收请求并开始处理它 
   2. 服务器的事件循环和队列中的所有任务将等待直到`time.sleep()`完成
      1. 服务器认为 `time.sleep()` 这不是一个I/O任务，所以它等待直到完成
      2. 服务器在等待期间不会接受任何新请求
   3. 然后，事件循环和队列中的所有任务将等待直到`service.get_pong`完成
      1. 服务器认为`service.get_pong()`这不是一个I/O任务，所以它等待直到完成
      2. 服务器在等待期间不会接受任何新请求
   4. 服务器返回响应。 
      1. 响应后，服务器开始接受新请求
2. `GET /good-ping` (def 定义的函数中含有阻塞IO操作)
   1. FastAPI 服务器接收请求并开始处理它
   2. FastAPI 将整个路由发送`good_ping`到线程池，工作线程将在线程池中运行该函数
   3. 在`good_ping`执行时，事件循环从队列中选择下一个任务并处理它们（例如接受新请求，调用数据库
      - 独立于主线程（即我们的 FastAPI 应用程序），工作线程将等待`time.sleep`完成，然后等待`service.get_pong`完成
      - 同步操作仅阻塞副线程，而不阻塞主线程。
   4. 完成工作后`good_ping`，服务器向客户端返回响应
3. `GET /perfect-ping` (async def 定义函数， 将IO阻塞操作转换为异步操作)
   1. FastAPI 服务器接收请求并开始处理它
   2. FastAPI 等待`asyncio.sleep(10)` 函数执行完毕，此时事件循环可以继续处理别的请求
   3. 事件循环从队列中选择下一个任务并处理它们（例如接受新请求，调用数据库）
   4. 完成后`asyncio.sleep(10)`，服务器转到下一行并等待`service.async_get_pong`
   5. 事件循环从队列中选择下一个任务并处理它们（例如接受新请求，调用数据库）
   6. 完成后`service.async_get_pong`，服务器向客户端返回响应

第二个需要注意的是，非阻塞等待或发送到线程池的操作必须是 I/O 密集型任务（例如打开文件、数据库调用、外部 API 调用）。
- CPU 密集型任务（例如繁重的计算、数据处理、视频转码）使用await是没有价值的，因为 CPU 必须工作才能完成任务，而 I/O 操作是外部的（如网络请求，读取数据库操作），服务器在等待操作完成时不执行任何操作，因此就可以进行接下来的任务了。
- 由于[GIL](https://realpython.com/python-gil/)的原因，在其他线程中运行 CPU 密集型任务也无效。简而言之，GIL 一次只允许一个线程工作，这使得它对于 CPU 任务毫无用处。
- 如果你想优化 CPU 密集型任务，你应该将它们发送给另外的进程，而不是同一个进程下的另外的线程。

** StackOverflow 上关于此问题的讨论 **
1. https://stackoverflow.com/questions/62976648/architecture-flask-vs-fastapi/70309597#70309597
   - 您也可以在这里查看[我的回答](https://stackoverflow.com/a/70309597/6927498)
2. https://stackoverflow.com/questions/65342833/fastapi-uploadfile-is-slow-compared-to-flask
3. https://stackoverflow.com/questions/71516140/fastapi-runs-api-calls-in-serial-instead-of-parallel-fashion

### 8. 定制自己的基础模型

拥有可控的全局基础模型能够让我们能够自定义应用程序内的所有模型。
例如，我们可以有一个标准的日期时间格式或为基本模型的所有子类添加一个父类方法。

```python
from datetime import datetime
from typing import Any
from zoneinfo import ZoneInfo

from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel, ConfigDict, model_validator


def convert_datetime_to_gmt(dt: datetime) -> str:
    if not dt.tzinfo:
        dt = dt.replace(tzinfo=ZoneInfo("UTC"))

    return dt.strftime("%Y-%m-%dT%H:%M:%S%z")


class CustomModel(BaseModel):
    model_config = ConfigDict(
        json_encoders={datetime: convert_datetime_to_gmt},
        populate_by_name=True,
    )

    @model_validator(mode="before")
    @classmethod
    def set_null_microseconds(cls, data: dict[str, Any]) -> dict[str, Any]:
        datetime_fields = {
            k: v.replace(microsecond=0)
            for k, v in data.items()
            if isinstance(k, datetime)
        }

        return {**data, **datetime_fields}

    def serializable_dict(self, **kwargs):
        """Return a dict which contains only serializable fields."""
        default_dict = self.model_dump()

        return jsonable_encoder(default_dict)

```
在上面的示例中，我们创建一个全局基础模型: 
- 将所有日期格式中的微秒降至 0
- 将所有日期时间字段序列化为具有显式时区的标准格式 

### 9. 文档
1. 除非您的 API 是公开的，否则默认隐藏文档。仅在选定的环境上显式显示它。

```python
from fastapi import FastAPI
from starlette.config import Config

config = Config(".env")  # 从.env 文件中解析出环境变量

ENVIRONMENT = config("ENVIRONMENT")  # 获取当前的环境名称
SHOW_DOCS_ENVIRONMENT = ("local", "staging")  # 明确定义可以显示文档的环境名称

app_configs = {"title": "My Cool API"}
if ENVIRONMENT not in SHOW_DOCS_ENVIRONMENT:
   app_configs["openapi_url"] = None  # 将openapi_url设置为None， 即不显示文档

app = FastAPI(**app_configs)
```
1. 帮助FastAPI生成易于理解的文档
   1. 设置`response_model`、`status_code`、`description`等
   2. 如果模型和状态不同，请使用responses路由属性为不同的响应添加文档
```python
from fastapi import APIRouter, status

router = APIRouter()

@router.post(
    "/endpoints",
    response_model=DefaultResponseModel,  # 默认pydantic响应模型 
    status_code=status.HTTP_201_CREATED,  # 默认 status code
    description="Description of the well documented endpoint",
    tags=["Endpoint Category"],
    summary="Summary of the Endpoint",
    responses={
        status.HTTP_200_OK: {
            "model": OkResponse, # 当状态为200是使用自定义 pydantic 模型
            "description": "Ok Response",
        },
        status.HTTP_201_CREATED: {
            "model": CreatedResponse,  #当状态为201是使用自定义pydantic 模型
            "description": "Creates something from user request ",
        },
        status.HTTP_202_ACCEPTED: {
            "model": AcceptedResponse,  # 当状态为202是使用自定义pydantic 模型
            "description": "Accepts request and handles it later",
        },
    },
)
async def documented_route():
    pass
```
将生成这样的文档:
![FastAPI Generated Custom Response Docs](images/custom_responses.png "Custom Response Docs")

### 10. 使用 Pydantic 的 BaseSettings 进行配置
Pydantic 提供了一个[强大的工具](https://docs.pydantic.dev/latest/concepts/pydantic_settings/)来解析环境变量并使用其验证器处理它们。
```python
from pydantic import AnyUrl, PostgresDsn
from pydantic_settings import BaseSettings  # pydantic v2

class AppSettings(BaseSettings):
    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"
        env_prefix = "app_"

    DATABASE_URL: PostgresDsn
    IS_GOOD_ENV: bool = True
    ALLOWED_CORS_ORIGINS: set[AnyUrl]
```
### 11. SQLAlchemy：设置数据库键命名约定
根据数据库的约定显式设置索引的命名比 sqlalchemy 的命名更可取。
```python
from sqlalchemy import MetaData

POSTGRES_INDEXES_NAMING_CONVENTION = {
    "ix": "%(column_0_label)s_idx",
    "uq": "%(table_name)s_%(column_0_name)s_key",
    "ck": "%(table_name)s_%(constraint_name)s_check",
    "fk": "%(table_name)s_%(column_0_name)s_fkey",
    "pk": "%(table_name)s_pkey",
}
metadata = MetaData(naming_convention=POSTGRES_INDEXES_NAMING_CONVENTION)
```
### 12. Migrations. Alembic.
1. 迁移必须是静态的且可恢复的。如果您的迁移依赖于动态生成的数据，那么请确保唯一动态的是数据本身，而不是其结构。
2. Generate migrations with descriptive names & slugs. Slug is required and should explain the changes.
3. Set human-readable file template for new migrations. We use `*date*_*slug*.py` pattern, e.g. `2022-08-24_post_content_idx.py`
```
# alembic.ini
file_template = %%(year)d-%%(month).2d-%%(day).2d_%%(slug)s
```
### 13. 设置数据库命名约定
与名称保持一致很重要。我们遵循的一些规则：
1. 小写蛇
2. 单数形式（例如`post`, `post_like`, `user_playlist`）
3. 使用模块前缀对相似的表进行分组，例如`payment_account` , `payment_bill`, `post`,`post_like`
4. 跨表保持一致，但具体命名是可以的，例如
   1. 在所有表中使用`profile_id`，但如果其中一些表只需要创建者的配置文件，请使用`creator_id`
   2. 用于`post_id`所有抽象表，如`post_like`, `post_view`，但在相关模块中使用具体命名，`course_id`如`chapters.course_id`
5. `_at`日期时间的后缀
6. `_date` 日期后缀

### 14. 设置异步测试客户端
使用数据库编写集成测试很可能会在将来导致混乱的事件循环错误。立即设置异步测试客户端，例如[async_asgi_testclient](https://github.com/vinissimus/async-asgi-testclient)或[httpx](https://github.com/encode/starlette/issues/652)
```python
import pytest
from async_asgi_testclient import TestClient

from src.main import app  # inited FastAPI app


@pytest.fixture
async def client():
    host, port = "127.0.0.1", "5555"
    scope = {"client": (host, port)}

    async with TestClient(
        app, scope=scope, headers={"X-User-Fingerprint": "Test"}
    ) as client:
        yield client


@pytest.mark.asyncio
async def test_create_post(client: TestClient):
    resp = await client.post("/posts")

    assert resp.status_code == 201
```
除非您有同步数据库连接（开玩笑？）或者不打算编写集成测试。
### 15. 后台任务 > asyncio.create_task

BackgroundTasks 可以[有效地运行](https://github.com/encode/starlette/blob/31164e346b9bd1ce17d968e1301c3bb2c23bb418/starlette/background.py#L25) 阻塞和非阻塞 I/O 操作，就像 FastAPI 处理阻塞路由一样（sync任务在线程池中运行，而async任务稍后等待）

- 不要对欺骗工作线程，也不要将阻塞 I/O 操作标记为`async`
- 不要将其用于繁重的 CPU 密集型任务。
```python
from fastapi import APIRouter, BackgroundTasks
from pydantic import UUID4

from src.notifications import service as notifications_service


router = APIRouter()


@router.post("/users/{user_id}/email")
async def send_user_email(worker: BackgroundTasks, user_id: UUID4):
    """Send email to user"""
    worker.add_task(notifications_service.send_email, user_id)  # send email after responding client
    return {"status": "ok"}
```
### 16. Typing 非常重要
FastAPI、Pydantic 和现代 IDE 鼓励使用类型提示。

**没有类型提示**

<img src="images/type_hintsless.png" width="400" height="auto">

**带类型提示**

<img src="images/type_hints.png" width="400" height="auto">

### 17. 以块的形式保存文件.
不要以为您的客户端会一直发送小文件
```python
import aiofiles
from fastapi import UploadFile

DEFAULT_CHUNK_SIZE = 1024 * 1024 * 50  # 50 megabytes

async def save_video(video_file: UploadFile):
   async with aiofiles.open("/file/path/name.mp4", "wb") as f:
     while chunk := await video_file.read(DEFAULT_CHUNK_SIZE):
         await f.write(chunk)
```
### 18.  小心动态 pydantic 字段 (Pydantic v1)
If you have a pydantic field that can accept a union of types, be sure the validator explicitly knows the difference between those types.

如果您有一个可以接收 union 类型的 pydantic 字段，请确保验证器明确知道这些类型之间的差异。
```python
from pydantic import BaseModel


class Article(BaseModel):
   text: str | None
   extra: str | None


class Video(BaseModel):
   video_id: int
   text: str | None
   extra: str | None

   
class Post(BaseModel):
   content: Article | Video

   
post = Post(content={"video_id": 1, "text": "text"})
print(type(post.content))
# OUTPUT: Article
# Article非常具有包容性，所有字段都是可选的，允许任何字典都有效
```
**解决方案:**
1. 验证仅允许输入有效字段，如果有未知的字段则引发错误
```python
from pydantic import BaseModel, Extra

class Article(BaseModel):
   text: str | None
   extra: str | None
   
   class Config:
        extra = Extra.forbid
       

class Video(BaseModel):
   video_id: int
   text: str | None
   extra: str | None
   
   class Config:
        extra = Extra.forbid

   
class Post(BaseModel):
   content: Article | Video
```
2. 如果字段很简单，请使用 Pydantic 的 Smart Union (>v1.9, <2.0)

如果字段很简单`int` 或者 `bool`，那么这是一个很好的解决方案，但它不适用于像类这样的复杂字段。

没有使用Smart Union

```python
from pydantic import BaseModel


class Post(BaseModel):
   field_1: bool | int
   field_2: int | str
   content: Article | Video

p = Post(field_1=1, field_2="1", content={"video_id": 1})
print(p.field_1)
# OUTPUT: True
print(type(p.field_2))
# OUTPUT: int
print(type(p.content))
# OUTPUT: Article
```
使用Smart Union

```python
class Post(BaseModel):
   field_1: bool | int
   field_2: int | str
   content: Article | Video

   class Config:
      smart_union = True


p = Post(field_1=1, field_2="1", content={"video_id": 1})
print(p.field_1)
# OUTPUT: 1
print(type(p.field_2))
# OUTPUT: str
print(type(p.content))
# OUTPUT: Article, because smart_union doesn't work for complex fields like classes
```

3. Fast Workaround

正确排序字段类型：从最严格的到宽松的

```python
class Post(BaseModel):
   content: Video | Article
```

### 19. SQL优先，Pydantic其次
- 通常，数据库处理数据的速度比 CPython 更快、更简洁
- 最好使用 SQL 来完成所有复杂的连接和简单的数据操作。
- 最好在数据库中聚合 JSON 以获取嵌套对象的响应.
```python
# src.posts.service
from typing import Mapping

from pydantic import UUID4
from sqlalchemy import desc, func, select, text
from sqlalchemy.sql.functions import coalesce

from src.database import database, posts, profiles, post_review, products

async def get_posts(
    creator_id: UUID4, *, limit: int = 10, offset: int = 0
) -> list[Mapping]: 
    select_query = (
        select(
            (
                posts.c.id,
                posts.c.type,
                posts.c.slug,
                posts.c.title,
                func.json_build_object(
                   text("'id', profiles.id"),
                   text("'first_name', profiles.first_name"),
                   text("'last_name', profiles.last_name"),
                   text("'username', profiles.username"),
                ).label("creator"),
            )
        )
        .select_from(posts.join(profiles, posts.c.owner_id == profiles.c.id))
        .where(posts.c.owner_id == creator_id)
        .limit(limit)
        .offset(offset)
        .group_by(
            posts.c.id,
            posts.c.type,
            posts.c.slug,
            posts.c.title,
            profiles.c.id,
            profiles.c.first_name,
            profiles.c.last_name,
            profiles.c.username,
            profiles.c.avatar,
        )
        .order_by(
            desc(coalesce(posts.c.updated_at, posts.c.published_at, posts.c.created_at))
        )
    )
    
    return await database.fetch_all(select_query)

# src.posts.schemas
import orjson
from enum import Enum

from pydantic import BaseModel, UUID4, validator


class PostType(str, Enum):
    ARTICLE = "ARTICLE"
    COURSE = "COURSE"

   
class Creator(BaseModel):
    id: UUID4
    first_name: str
    last_name: str
    username: str


class Post(BaseModel):
    id: UUID4
    type: PostType
    slug: str
    title: str
    creator: Creator

    @validator("creator", pre=True)  # before default validation
    def parse_json(cls, creator: str | dict | Creator) -> dict | Creator:
       if isinstance(creator, str):  # i.e. json
          return orjson.loads(creator)

       return creator
    
# src.posts.router
from fastapi import APIRouter, Depends

router = APIRouter()


@router.get("/creators/{creator_id}/posts", response_model=list[Post])
async def get_creator_posts(creator: Mapping = Depends(valid_creator_id)):
   posts = await service.get_posts(creator["id"])

   return posts
```

如果 DB 的聚合数据是一个简单的 JSON，那么我们可以使用 Pydantic 的Json字段类型，它会首先加载原始 JSON。
```python
from pydantic import BaseModel, Json

class A(BaseModel):
    numbers: Json[list[int]]
    dicts: Json[dict[str, int]]

valid_a = A(numbers="[1, 2, 3]", dicts='{"key": 1000}')  # becomes A(numbers=[1,2,3], dicts={"key": 1000})
invalid_a = A(numbers='["a", "b", "c"]', dicts='{"key": "str instead of int"}')  # raises ValueError
```

### 20. 验证主机（如果用户可以发送公开可用的 URL）
例如，我们有一个特定的入口:
1. 接受来自用户的媒体文件,
2. 为该文件生成唯一的 url,
3. 返回 url 给用户,
   1. 他们将在其他端点中使用它们，例如`PUT /profiles/me`，`POST /posts`
   2. 这些入口仅接受来自白名单主机的文件
4. 使用此名称和匹配的 URL 将文件上传到 AWS。

如果我们不将 URL 主机列入白名单，那么不良用户将有机会上传危险链接
```python
from pydantic import AnyUrl, BaseModel

ALLOWED_MEDIA_URLS = {"mysite.com", "mysite.org"}

class CompanyMediaUrl(AnyUrl):
    @classmethod
    def validate_host(cls, parts: dict) -> tuple[str, str, str, bool]:  # pydantic v1
       """Extend pydantic's AnyUrl validation to whitelist URL hosts."""
        host, tld, host_type, rebuild = super().validate_host(parts)
        if host not in ALLOWED_MEDIA_URLS:
            raise ValueError(
                "Forbidden host url. Upload files only to internal services."
            )

        return host, tld, host_type, rebuild


class Profile(BaseModel):
    avatar_url: CompanyMediaUrl  # only whitelisted urls for avatar

```
### 21. 在自定义pydantic 验证器中直接触发 ValueError

它将向用户返回一个详细的响应。
```python
# src.profiles.schemas
from pydantic import BaseModel, validator

class ProfileCreate(BaseModel):
    username: str
    
    @validator("username")  # pydantic v1
    def validate_bad_words(cls, username: str):
        if username  == "me":
            raise ValueError("bad username, choose another")
        
        return username


# src.profiles.routes
from fastapi import APIRouter

router = APIRouter()


@router.post("/profiles")
async def get_creator_posts(profile_data: ProfileCreate):
   pass
```
**Response Example:**

<img src="images/custom_bad_response.png" width="400" height="auto">

### 22. FastAPI将Pydantic对象转换为dict，然后转换为Pydantic对象，然后转换为JSON
If you think you can return Pydantic object that matches your route's `response_model` to make some optimizations,
then it's wrong. 

使用您认为直接返回Pydantic 对象，并且在路由装饰器中定义`response_model` 来优化输出，那么这种想法是错误的。

首先 FastAPI 调用pydantic 对象的`jsonable_encoder`方法，将其转换为dict，之后再将dict变量转换为`response_model`中设置的模型，然后才将您的对象序列化为 JSON。
 
```python
from fastapi import FastAPI
from pydantic import BaseModel, root_validator

app = FastAPI()


class ProfileResponse(BaseModel):
    @root_validator
    def debug_usage(cls, data: dict):
        print("created pydantic model")

        return data

    def dict(self, *args, **kwargs):
        print("called dict")
        return super().dict(*args, **kwargs)


@app.get("/", response_model=ProfileResponse)
async def root():
    return ProfileResponse()
```
**日志输出:**
```
[INFO] [2022-08-28 12:00:00.000000] created pydantic model
[INFO] [2022-08-28 12:00:00.000010] called dict
[INFO] [2022-08-28 12:00:00.000020] created pydantic model
[INFO] [2022-08-28 12:00:00.000030] called dict
```
### 23. 如果必须使用sync SDK，那么在线程池中运行它.

外部服务交互过程中，如果必须使用第三方库，但事实这个库并不是 async 的，那么请在额外的工作线程中进行 HTTP 访问调用。

举一个简单的例子，我们可以使用我们熟知的`run_in_threadpoolstarlette`。

```python
from fastapi import FastAPI
from fastapi.concurrency import run_in_threadpool
from my_sync_library import SyncAPIClient 

app = FastAPI()


@app.get("/")
async def call_my_sync_library():
    my_data = await service.get_my_data()

    client = SyncAPIClient()
    await run_in_threadpool(client.make_request, data=my_data)
```
### 24. Use linters (black, ruff)
With linters, you can forget about formatting the code and focus on writing the business logic.

Black is the uncompromising code formatter that eliminates so many small decisions you have to make during development.
Ruff is "blazingly-fast" new linter that replaces autoflake and isort, and supports more than 600 lint rules.

It's a popular good practice to use pre-commit hooks, but just using the script was ok for us.
```shell
#!/bin/sh -e
set -x

ruff --fix
black src tests
```
### Bonus Section
Some very kind people shared their own experience and best practices that are definitely worth reading.
Check them out at [issues](https://github.com/zhanymkanov/fastapi-best-practices/issues) section of the project.

例如，[lowercase00](https://github.com/zhanymkanov/fastapi-best-practices/issues/4) 详细描述了他们使用权限和身份验证、基于类的服务和视图、任务队列、自定义响应序列化器、dynaconf 配置等的最佳实践。

如果您有任何关于使用 FastAPI 的经验想要分享，无论是好还是坏，都非常欢迎您创建一个新问题。我们很高兴阅读它。
