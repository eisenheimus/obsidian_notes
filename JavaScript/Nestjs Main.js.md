``` typescript

//main.ts *********************************************************************************************************************

import { NestFactory } from '@nestjs/core';

import { AppModule } from '@modules/common/app/app.module';

import { Logger, NestApplicationOptions, ValidationPipe } from '@nestjs/common';

import { ConfigService } from '@nestjs/config';

import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';

import { CorsOptions } from '@nestjs/common/interfaces/external/cors-options.interface';

import { NestExpressApplication } from '@nestjs/platform-express';

import { json, urlencoded } from 'express';

  

  

async function bootstrap() {

  

  

  const corsConfig: CorsOptions = {

    origin: true,

    exposedHeaders: ['origin', 'x-access-token', 'authorization', 'Authorization', 'content-type', 'Set-cookie', 'Cookies', 'Cookie'],

    allowedHeaders: ['origin', 'content-type', 'Access-Control-Allow-Origin', 'Cookies', 'Cookie'],

    methods: ['PUT', 'GET', 'POST', 'DELETE', 'OPTIONS', 'PATCH'],

    maxAge: 1800,

    credentials: true,

  };

  const appConfig: NestApplicationOptions = {

    cors: corsConfig,

  };

  

  

  const app = await NestFactory.create<NestExpressApplication>(AppModule, appConfig);

  const configService = app.get(ConfigService);

  const port = configService.get('SERVER_PORT');

  const prefix = configService.get('SERVER_PREFIX');

  

  

  app.enableCors();

  app.setGlobalPrefix(prefix);

  // limit: '50mb' увеличивает возможный объем данных с клиента на сервер

  app.use(json({ limit: '50mb' }));

  app.use(urlencoded({ extended: true, limit: '50mb' }));

  const config = new DocumentBuilder().setTitle('test').setVersion('testV').build();

  const document = SwaggerModule.createDocument(app, config);

  const logger = new Logger('MAIN')

  SwaggerModule.setup('/swagger', app, document);

  

  

  await app.listen(port);

  

  

  logger.verbose(`*** Server is running on port: ${port} with prefix: /${prefix} ***`)

}

bootstrap();

```

  
  

//app.modules.ts *********************************************************************************************************************

@Module({

  imports: [

    UsersCredentialsModule,

    UsersModule,

    LoggerModule,

    ConfigModule.forRoot(config),

    TypeOrmModule.forRootAsync(RMAP_CONFIG),

    TypeOrmModule.forRootAsync(UDS_CONFIG),

    ScheduleModule.forRoot(),

    SmRequestsModule,

  ],

  controllers: [

    SmRequestsController,

    AppController,

  ],

  providers: [

    AppService,

    {

      provide: APP_GUARD,

      useClass: AuthGuard('sudir'),

    },

  ],

})

export class AppModule { }

  
  

  

//sudir.strategy.ts *********************************************************************************************************************

@Injectable()

export class SudirStrategy extends PassportStrategy(Strategy, 'sudir'){

    constructor(

        private readonly jwtService: JwtService,

        private readonly loggerService: LoggerService,

    ){

        super();

    }

  

  

    async validate (req: Request)  {

  

  

        if(!req?.headers) return true;

  

  

        const authorization: string = req.headers?.authorization;

        const token: string = authorization?.replace('Bearer', '').trim();

        let login: string = null;

        let pdi = null;

  

  

        try {

            const payload = await this.jwtService.decode(token);

            login = payload?.['sub'];

            pdi = payload?.['sberpdi'];

  

  

            this.loggerService.verbose(`SudirStrategy: Аутентификация пользователя ${login} ${token}`)

  

  

        } catch (e) {

            this.loggerService.error(`SudirStrategy: ошибка валидации токена СУДИР ${token}`)

  

  

            throw new UnauthorizedException();

        }

        return { login, pdi }

    }

}

  

  

//token.strategy.ts *********************************************************************************************************************

import {ForbiddenException, Logger, Req, UnauthorizedException } from "@nestjs/common";

import { PassportStrategy } from "@nestjs/passport";

import { Strategy} from "passport-custom";

import { Injectable } from "@nestjs/common/decorators/core";

import { JwtService } from "@nestjs/jwt";

import { Request } from "express";

import { LoggerService } from "@modules/common/logger/logger.service";

import { UserPayloadInterface } from "../interfaces/userPaylod.interface";

  
  

  

@Injectable()

export class TokenStrategy extends PassportStrategy(Strategy, 'token'){

    constructor(

        private readonly jwtService: JwtService,

        private readonly loggerService: LoggerService,

    ){

        super();

    }

    async validate(req: Request){

  

  

        const login: string = req.user['login'];

        const tokenName: string = 'x-access-token'

        const token: string = req.headers?.[tokenName]?.toString();

        const regExp: RegExp = new RegExp(/\/login$/);

  

  

        if(regExp.test(req.url)) {

            this.loggerService.log(`TokenStrategy(): Запрос роута ${req.url} клиентом с логином ${login} `);

            return {login}

        }

  

  

        if(!token) {

            this.loggerService.error(`TokenStrategy(): Запрос роута ${req.url} клиентом с логином ${login} без ${tokenName}`);

            throw new UnauthorizedException(`Не передан ${tokenName}`);

        }

  

  

        const payload: string | { [key: string]: string } | UserPayloadInterface = this.jwtService.decode(token);

  

  

        if(!payload['isActive']) {

            this.loggerService.error(`TokenStrategy(): Запрос роута ${req.url} клиентом с логином ${login} имеющим активный БАН`);

            throw new ForbiddenException('Пользователь имеет БАН');

        }

  

  

        this.loggerService.log(`TokenStrategy(): Запрос роута ${req.url} клиентом с логином ${login} и токеном ${token}`);

  

  

        return payload;

    }  

  

  

}

  
  

  

//auth.module.ts *********************************************************************************************************************

@Module({

    imports: [

      ConfigModule,

      JwtModule.registerAsync({

        useFactory: (configService: ConfigService) => {

          return {

              secret: configService.get('SERVER_JWT_SECRET'),

              signOptions: {

                  expiresIn: configService.get('SERVER_JWT_EXP') || '5m'

              }

          }

        },

        inject: [ConfigService]

      }),

      forwardRef(() => UsersModule ),

      SudirRolesModule,

  

  

    ],      

    exports: [AuthModule],

    providers: [AuthService, SudirStrategy, JwtStrategy, TokenStrategy, WsStrategy, Messages],

    controllers: [AuthController]

  

  

})

  

  

export class AuthModule {}

  
  

  

// auth.service.ts *********************************************************************************************************************

import { InternalServerErrorException, Injectable, Logger, UnauthorizedException, ForbiddenException, NotFoundException, NotAcceptableException } from "@nestjs/common";

import { ConfigService } from "@nestjs/config";

import { JwtService } from "@nestjs/jwt";

import { JwtTokenPayloadType } from "./common/types/jwtTokenPayload.type";

import { JwtToken, JwtTokenPayloadInterface, secretOptionsInterface } from "./common/interfaces";

import { UsersService } from "../users/modules/users/users.service";

import { UsersEntity } from "../users/modules/users/users.entity";

import { LoggerService } from "../logger/logger.service";

import { SudirRolesService } from "../sudirRoles/sudirRoles.service";

import { GetUserInfoInterface, SudirRolesUserInterface } from "../sudirRoles/sudirRoles.type";

  
  

  

@Injectable()

export class AuthService{

  

  

    constructor(

        private readonly jwtService: JwtService,

        private readonly configService: ConfigService,

        private readonly userService: UsersService,

        private readonly loggerService: LoggerService,

        private readonly sudirRolesService: SudirRolesService,

        ){}

  

  

    private readonly secretOptions: secretOptionsInterface = {

        secret: this.configService.get<string>('SERVER_JWT_SECRET'),

            expiresIn: this.configService.get<string>('SERVER_JWT_EXP') || '5m'

    }

    async generateJwtToken({login, pdi} : {login: string, pdi: string}): Promise<JwtToken & JwtTokenPayloadInterface> {

        if(!login) throw new UnauthorizedException();        

  

  

        this.loggerService.warn(`generateJwtToken(): Запуск метода сервиса login: ${login}, pdi: ${pdi} `);

  

  

        let userDataDk: UsersEntity | null;

        let userDataSudir: GetUserInfoInterface = null;

        let dkRoles: string[] = [];

        let sudirRoles: string[] = [];

  

  

        try {

            if(!pdi) throw new NotAcceptableException(`Отсутствует pdi у пользователя ${login}`)

  

  

            userDataSudir = await this.sudirRolesService.getUserInfo(pdi);

            if(!userDataSudir) throw new NotFoundException(`Пользователя ${login} с pdi ${pdi} не найден в системе СУДИР`);

  

  

            sudirRoles = userDataSudir?.roles || [];

            this.loggerService.log(`generateJwtToken(): Сформирован список ролей пользователя ${login} из СУДИР  - ${sudirRoles}`)

        } catch (error) {

            this.loggerService.error(`generateJwtToken(): Ошибка получения данных из СУДИР - ${error.message}`);

        }

  

  

        try{

            userDataDk = await this.userService.findUserByLogin(login);

            if(!userDataDk) {  

                const USER_TUZ=this.configService.get<string>('ARM_ZNO_TUZ') || 'ИСО МЦТП';

                const USERNAME_PLACEHOLDER=this.configService.get<string>('АRM_ZNO_USERNAME_PLACEHOLDER') || 'auto';

  

  

                if(userDataSudir?.userName && userDataSudir?.roles && userDataSudir?.roles.some(role => /^RMAP_(USER|MODERATOR)_(ARM|LITE|SCS|EIRS|POS)$/.test(role?.trim()))) this.userService.create(Object.assign(new UsersEntity, {login: userDataSudir.userName, fullName: USERNAME_PLACEHOLDER, smContact: USER_TUZ}));

                throw new NotFoundException(`Пользователь с логином ${login} не найден в системе ДК`);

            }

  

  

            dkRoles = (userDataDk?.userCredentials?.map( r => r.credentialName )).filter(role => role) ?? [];

            this.loggerService.log(`generateJwtToken(): Сформирован список ролей пользователя ${login} из ДК - ${dkRoles}`)

        }

        catch(error){

            this.loggerService.error(`generateJwtToken(): Ошибка получения данных из ДК - ${error.message}`);

        }

        const payload: JwtTokenPayloadInterface = {

            login,

            roles: [...sudirRoles, ...dkRoles],

            isActive: true, // TODO: После полного перехода на СУДИР, заменить кодом: Boolean(userDataSudir?.active),

            fullName: userDataDk?.fullName,

            smName: userDataDk?.smName,

            smContact: userDataDk?.smContact,

            pdi

        }

  

  

        try {

            this.loggerService.log(`generateJwtToken(): Сформирован payload ${JSON.stringify(payload)}`)

        }

        catch (error) {

            this.loggerService.error(`generateJwtToken(): Ошибка сериализации payload - ${error?.message}`)

        }

        try{            

            const token: string = this.jwtService.sign(payload);

  

  

            this.loggerService.log(`generateJwtToken(): Сформирован токен пользователя: ${token}`)

  

  

            return { token, ...payload };

        }

        catch(error){

            this.loggerService.error(`generateJwtToken(): Не удалось сгенерировать токен: ${error.message}`);

            throw new InternalServerErrorException('Не удалось сгенерировать токен')

        }

    }

  

  

    decodeJwtToken(token: string): JwtTokenPayloadType {

        this.loggerService.log(`decodeJwtToken(): Попытка декодировать токен ${token}`);

  

  

        return this.jwtService.decode(token);

    }

  

  

    verifyJwtToken(token): {isValidToken: boolean, message: string} | (JwtTokenPayloadInterface & {isValid: boolean }) {

        let payload;

        let isValidToken = false;

        let verified: object = {};

        let message;

  

  

        try {

            payload = this.jwtService.decode(token);

            verified = this.jwtService.verify(token, this.secretOptions);

        }

        catch(error){

            message = error;

            this.loggerService.error(`verifyJwtToken(): токен не прошел валидацию ${error.message}`);

        }

        finally{

            return !payload?.login

                ? {isValidToken, message}

                : {...payload, isValidToken: Boolean(verified?.['login'])};

  

  

        }

    }

  
  

  

}

  
  

  

//socket.module.ts *********************************************************************************************************************

@Module({

  providers: [SocketEventsGateway],

  imports: [ArmMainModule, ImporterModule],

})

export class SocketEventsModule { }

  
  

  

//socketsEvents.gateway.ts *********************************************************************************************************************

@WebSocketGateway({

  cors: {

    origin: '*'

  },

  path: '/roadmap/api/ws/'

})

  

  

export class SocketEventsGateway implements OnGatewayConnection {

  constructor(

    private readonly armMainService: ArmMainService,

    private readonly importerService: ImporterService

  ) { }

  @WebSocketServer() server: Server;

  handleConnection(client: any, ...args: any[]) {

    console.log('WebSocketServer connection');

  }

  

  

  @SubscribeMessage(GatewayEvents.fullImportExcelFile)

  @UseGuards(new WsRolesGuard([RolesEnum.RMAP_MODERATOR_ARM, RolesEnum.RMAP_MODERATOR_LITE]))

  async fullImportExcelFile(

    @ConnectedSocket() socket: Socket,

    @MessageBody() importDto: FullImportDtoInterface,

  ) {

    return await this.importerService.fullImportExcelFile(importDto);

  }

  

  

  //getLogin.decorator.ts *********************************************************************************************************************

  import { createParamDecorator, ExecutionContext } from '@nestjs/common';

  

  

export const GetLogin = createParamDecorator(

  (data: unknown, ctx: ExecutionContext) => {

    const request = ctx.switchToHttp().getRequest();

  

  

    return request.user.login ;

  },

);

  

  

//roles.decorator.ts *********************************************************************************************************************

import { SetMetadata } from '@nestjs/common';

import {RolesEnum} from '../enums/roles.enum'

  

  

export const Roles = (...roles: RolesEnum[]) => {

  return  SetMetadata('roles', roles);

}

  

  

//@UseGuards(new RestRolesGuard(['ARM', 'LITE', RolesEnum.RMAP_MODERATOR_ARM, RolesEnum.RMAP_MODERATOR_LITE]))

// restRoles.guard.ts *********************************************************************************************************************

@Injectable()

export class RestRolesGuard implements CanActivate{

    constructor(

            private roles: string[],

        ){}

  

  

    canActivate(context: ExecutionContext) {

        const jwtService: JwtService = new JwtService()

        const req: Request = context.switchToHttp().getRequest();

        const token: string = (req.headers?.['x-access-token']?? '')?.toString();

        const logger = new Logger();

  

  

        logger.warn('token', token);

  

  

        if(!token)  {

            Logger.error('RestRestRolesGuard: Ошибка валидации x-access-token');

            return false

        }

  

  

        const payload: JwtTokenPayloadType = jwtService.decode(token);

        const isAllowed: boolean = this.roles.some((role) =>  payload['roles'].includes(role));

        const isManager: boolean = payload['roles']?.includes(RolesEnum.RMAP_MANAGER);

  

  

        req.headers['login'] = payload?.['login'];

        if(isAllowed || isManager) {

            logger.warn(`RestRolesGuard: доступ пользователю ${payload?.['login']} РАЗРЕШЕН ${ payload['roles']}`)

            return true

        } else {

  

  

            logger.error(`RestRolesGuard: доступ пользователю ${payload?.['login']} ЗАПРЕЩЕН ${ payload['roles']}`)

  

  

            return false;

        }

    }

}

  

  

//wsRoles.guard.ts *********************************************************************************************************************

@Injectable()

export class WsRolesGuard implements CanActivate{

    constructor(

            private roles: string[],

        ){}

  

  

    canActivate(context: ExecutionContext) {

  

  

        const jwtService: JwtService = new JwtService()

        const client =  context.switchToWs().getClient()?.handshake?.headers;

        const token: string = context.switchToWs().getClient()?.handshake?.headers?.['x-access-token'];

        const logger = new Logger();

  

  

        logger.warn('token', token);

        if(!token)  {

            Logger.error('WsRolesGuard: Ошибка валидации x-access-token');

            return false

        }

        const payload: JwtTokenPayloadType = jwtService.decode(token)

        const isAllowed: boolean = this.roles.some((role) => payload['roles']?.includes(role));

        const isManager: boolean = payload['roles']?.includes(RolesEnum.RMAP_MANAGER);

  

  

        client['login'] = payload?.['login'];

  

  

        if(isAllowed || isManager) {

            logger.warn(`RestRolesGuard: доступ пользователю ${payload?.['login']} РАЗРЕШЕН ${ payload['roles']}`)

            return true

        } else {

  

  

            logger.error(`RestRolesGuard: доступ пользователю ${payload?.['login']} ЗАПРЕЩЕН ${ payload['roles']}`)

  

  

            return false;

        }

    }

}

  

  

//configModule.ts *********************************************************************************************************************

import { ConfigModule } from "@nestjs/config"

import { join } from "path"

  

  

export const config: ConfigModule = ({

    isGlobal: true,

    envFilePath: [join('/', 'secman', 'config.env'), '.env']

})

  

  

//orm.config.ts

function rmapParams(configService: ConfigService): TypeOrmModuleOptions {

    return {

        name: 'RMAP',

        type: 'postgres',

        schema: configService.get<string>('RMAP_DATABASE_SCHEMA'),

        host: configService.get<string>('RMAP_DATABASE_HOST'),

        port: +configService.get<string>('RMAP_DATABASE_PORT'),

        database: configService.get<string>('RMAP_DATABASE_NAME'),

        username: configService.get<string>('RMAP_DATABASE_USERNAME'),

        password: configService.get<string>('RMAP_DATABASE_PASSWORD'),

        ssl: false,

        autoLoadEntities: false,

        cache: true,

        synchronize: true,            

        retryAttempts: 1,

        entities: armEntities,

        migrations: [

            join('./', 'src', 'migrations', '*.{ts,js}'),

        ],

        extra: {

            max: +configService.get<number>('MAX_POOL') || 10, // Максимальное количество подключений

            idleTimeoutMillis: +configService.get<number>('POOL_CLOSE_CONNECTION_TIMEOUT_MS') || 30000, // Время простоя перед закрытием подключения (в миллисекундах)

            connectionTimeoutMillis: +configService.get<number>('POOL_CONNECTION_TIMEOUT_MS') || 5000, // Время ожидания подключения (в миллисекундах)

        },

    }

}

  

  

export const RMAP_CONFIG:  TypeOrmModuleAsyncOptions = ({

    inject: [ConfigService],

    useFactory: (configService: ConfigService): TypeOrmModuleAsyncOptions => {

        (new Logger('ORM CONFIG')).warn('*** Connected to RMAP ***');

  

  

        return rmapParams(configService);

    }

});

  

  

export const UDS_CONFIG:  TypeOrmModuleAsyncOptions = ({

    inject: [ConfigService],

    useFactory: (configService: ConfigService): TypeOrmModuleAsyncOptions => {

        const TO: string = !(configService.get('IS_DISCONNECT_UDS') === 'true') ? 'UDS' : 'RMAP';

        const UDS: TypeOrmModuleOptions = {

            name: 'UDS',

            type: 'postgres',

            schema: configService.get<string>('UDS_DATABASE_SCHEMA'),

            host: configService.get<string>('UDS_DATABASE_HOST'),

            port: +configService.get<string>('UDS_DATABASE_PORT') || 5433,

            database: configService.get<string>('UDS_DATABASE_NAME'),

            username: configService.get<string>('UDS_DATABASE_USERNAME'),

            password: configService.get<string>('UDS_DATABASE_PASSWORD'),

            ssl: false,

            cache: true,

            synchronize: false,

            retryAttempts: 1,

            entities:udsEntities,

            migrations: [

                join('./', 'src', 'migrations', '*.{ts,js}'),

            ],

            logging: Boolean(configService.get<string>('ORM_LOGGING')),

            logger: 'file'

        };

  

  

        (new Logger('ORM CONFIG')).warn(`*** Connected to ${TO} ***`);

        return TO === 'UDS' ? UDS : rmapParams(configService);

    }

});

  

  

//orm.dataSource.ts *********************************************************************************************************************

export const createDataSource = (configserive: ConfigService): DataSource =>{

  

  

    return configserive.get<string>('DB_TYPE') === 'SQLITE'

        ?

            new DataSource({

                type: 'sqljs',

                location:  './dev.db',

                synchronize: true,

                autoSave: true,

                entities: [

                    join('server', 'src', 'modules', '*.{ts,js}'),

                ],

                migrations: [

                    join('server', 'src', 'migrations', '*.{ts,js}'),

                ],

            })

  

  

        :

            new DataSource({

                schema: 'public',

                type: 'postgres',

                host: configserive.get<string>('DATABASE_HOST'),

                port: 5433,

                database: configserive.get<string>('DATABASE_NAME'),

                username: configserive.get<string>('usernameDB'),

                password: configserive.get<string>('passwordDB'),

  

  

            })

  

  

}

  

  

//scheduler.module.ts  *********************************************************************************************************************

@Module({

  imports: [

    TypeOrmModule.forFeature([SchedulerEntity]),

    ArmMainModule,

    SmRequestsModule,

    SmRequestsForNewDateModule,

    KeywordsModule,

    RtrModule,

    NotificationsModule,

    ConfigModule,

    BackupTablesModule,

  ],

  controllers: [],

  providers: [SchedulerHelper, SchedulerService ],

  exports: [],

})

export class SchedulerModule {}

  

  

//SchedulerHelper.ts

@Injectable()

export class SchedulerHelper {

  constructor(

    @InjectRepository(SchedulerEntity)

    private readonly schedulerRepository: Repository<SchedulerEntity>,

    private schedulerRegistry: SchedulerRegistry

  ) {

    this.checkJobs();

  }

  private async checkJobs() {

    const schedulerJobs = await this.schedulerRepository.find();

        if(!schedulerJobs.length) return;

    schedulerJobs.forEach((job) =>{

      try{

        const cronJob = this.schedulerRegistry.getCronJob(job.jobName);

        cronJob.setTime(new CronTime(job.startTime))

            if(job.isEnabled) {

            cronJob.start();

          }

            }catch(e){

                console.error(`Ошибка! Не найден CronJob с именем ${job.jobName}`);

            }

        })

  }

  

  

}

  

  

//  @Cron('* * * * *', { name: 'automaticReturnArm', disabled: true })

  

  

//OneToMany *********************************************************************************************************************

@Entity({ name: 'arm_main' })

export class ArmMainEntity extends BaseEntity{

    // Дата создания записи в таблицу arm_main

    @CreateDateColumn({nullable: true})

    createdAt: Date;

  

  

    // Дата обновления записи в таблице arm_main

    @UpdateDateColumn()    

    updatedAt: Date;

  

  

    @OneToMany((type) => LoggerArmMainEntity, (loggerArmMainEntity) => loggerArmMainEntity.armMainEntity, {onDelete: 'SET NULL'})

    loggerArmMainEntity: LoggerArmMainEntity[];

  

  

    @Column({type: 'int', nullable: true, default: null})

    refreshSmRequestsId: number;

  

  

    @Column({type: 'int', nullable: true, default: null})

    pfSmRequestsId: number;

}

  

  

@Entity({name: 'logger_arm_main'})

export class LoggerArmMainEntity {

    @PrimaryGeneratedColumn()

    id: number;

  

  

    // Предыдущее значения измененного поля

    @Column({ nullable: true})

    oldValue: string;

  

  

    // Новое значение измененного поля

    @Column({ nullable: true})

    newValue: string;

  

  

    // ФИО сотрудника, внесшего изменение

    @Column({nullable: true})

    fullName: string;

  

  

    // Дата создания

    @CreateDateColumn({nullable: true})

    createdAt: Date;

    // Тип действия

    @Column({nullable: false})

    actionType: string;

  

  

    // Связанная таблица с логами. Параметр 'SET NULL' зануляет значение armMainEntityId при удалении записи из main, а не удаляет как 'CASCASE'

    @ManyToOne((type) => ArmMainEntity, (armMainEntity) => armMainEntity.loggerArmMainEntity, {onDelete: 'SET NULL'})

    armMainEntity: ArmMainEntity

}

  

  

// TypePRM requests *********************************************************************************************************************

    const armMainEntities: ArmMainEntity[] = await this.crudFindWithParam(

        {where: {id: In(ids)},

        relations: {loggerArmMainEntity: true}

        }

      );

  

  

    const armMainEntities: ArmMainEntity[] = await this.crudFindWithParam({

        where: [

          {

          newSerial: '',

          refreshSolutionText: Like('%ARM010202%'),

          refreshDateEnd: Not(IsNull())

        }]});

  

  

    return await this.armMainRepository.find({

      where: {

        id: In(ids)

      }

    })

  

  

//crud.class.ts *********************************************************************************************************************

@Injectable()

export class CRUD<T> {

  

  

    constructor(

        @Inject(Repository<T>)

        protected repository: Repository<T>,

    ){

    }

    private logger = new Logger();

    async findOne(options: FindOneOptions): Promise<T>{

        try{

            return await this.repository.findOne(options);

        }

        catch(error){

            this.logger.error(`CRUD.findOne(${this.constructor.name}):  ${error.message}`);

        }

    }

  

  

    async find(options?: FindManyOptions): Promise<T[]>{

        try{

            return await this.repository.find(options);

        }

        catch(error){

            this.logger.error(`CRUD.find(${this.constructor.name}):  ${error.message}`);

        }

    }

  

  

    async delete(id: number): Promise<DeleteOptions> {

        try {

            return this.repository.delete(id)

        }

        catch (error) {

            this.logger.error(`CRUD.delete(${this.constructor.name}):  ${error.message}`);

        }

    }

  

  

    async save(entity: T | T[]): Promise<T | T[]> {

        try {

            return await (Array.isArray(entity) ? this.repository.save(entity): this.repository.save(entity));

        }

        catch (error) {

            this.logger.error(`CRUD.save(${this.constructor.name}):  ${error.message}`);

        }

    }

}

  

  

//.eslint.js *********************************************************************************************************************

    module.exports = {

  parser: '@typescript-eslint/parser',

  parserOptions: {

    project: 'tsconfig.json',

    tsconfigRootDir: __dirname,

    sourceType: 'module',

  },

  plugins: ['@typescript-eslint/eslint-plugin'],

  extends: [

    'plugin:@typescript-eslint/recommended',

    'plugin:prettier/recommended',

  ],

  root: true,

  env: {

    node: true,

    jest: true,

  },

  ignorePatterns: ['.eslintrc.js'],

  rules: {

    '@typescript-eslint/interface-name-prefix': 'off',

    '@typescript-eslint/explicit-function-return-type': 'off',

    '@typescript-eslint/explicit-module-boundary-types': 'off',

    '@typescript-eslint/no-explicit-any': 'off',

  },

};

  

  

//.pretirerrc *********************************************************************************************************************

{

  "singleQuote": true,

  "trailingComma": "all"

}

  

  

//package.json *********************************************************************************************************************

  

  

{

  "name": "backend",

  "version": "1.17.0",

  "description": "release",

  "author": "",

  "private": true,

  "license": "UNLICENSED",

  "scripts": {

    "build": "node build.mjs",

    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",

    "start": "node build.mjs --dev",

    "dev": "node build.mjs --dev --watch",

    "debug": "node --inspect --require ts-node/register src/main.ts",

    "start:dev": "node build.mjs --dev --watch",

    "start:debug": "node --inspect --require ts-node/register src/main.ts",

    "start:prod": "node dist/main.js",

    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",

    "test": "jest",

    "test:watch": "jest --watch",

    "test:cov": "jest --coverage",

    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",

    "test:e2e": "jest --config ./test/jest-e2e.json"

  },

  "dependencies": {

    "@nestjs/axios": "3.0.2",

    "@nestjs/cache-manager": "3.0.0",

    "@nestjs/common": "10.0.0",

    "@nestjs/config": "3.0.0",

    "@nestjs/core": "10.0.0",

    "@nestjs/event-emitter": "2.0.2",

    "@nestjs/jwt": "10.1.0",

    "@nestjs/passport": "10.0.0",

    "@nestjs/platform-express": "10.0.0",

    "@nestjs/platform-socket.io": "10.1.3",

    "@nestjs/schedule": "4.0.0",

    "@nestjs/swagger": "7.1.6",

    "@nestjs/typeorm": "10.0.0",

    "@nestjs/websockets": "10.1.3",

    "axios": "1.8.4",

    "cache-manager": "6.4.0",

    "class-transformer": "0.5.1",

    "class-validator": "0.14.0",

    "cookie-parser": "1.4.6",

    "date-fns": "2.30.0",

    "exceljs": "4.3.0",

    "joi": "17.11.0",

    "nestjs-cacheable": "1.0.0",

    "passport": "0.6.0",

    "passport-custom": "1.1.1",

    "passport-jwt": "4.0.1",

    "pg": "8.11.2",

    "reflect-metadata": "0.1.13",

    "rxjs": "7.8.1",

    "socket.io": "4.7.2",

    "sql.js": "1.8.0",

    "typeorm": "0.3.17",

    "uuid": "9.0.0"

  },

  "devDependencies": {

    "@es-exec/esbuild-plugin-start": "0.0.5",

    "@kang-heewon/esbuild-plugin-typescript-decorators": "0.1.0",

    "@nestjs/schematics": "10.0.0",

    "@nestjs/testing": "10.0.0",

    "@types/express": "4.17.17",

    "@types/jest": "29.5.2",

    "@types/node": "20.3.1",

    "@types/passport-jwt": "3.0.9",

    "@types/supertest": "2.0.12",

    "@types/uuid": "9.0.2",

    "@typescript-eslint/eslint-plugin": "5.59.11",

    "@typescript-eslint/parser": "5.59.11",

    "esbuild": "0.19.4",

    "eslint": "8.42.0",

    "eslint-config-prettier": "8.8.0",

    "eslint-plugin-prettier": "4.2.1",

    "jest": "29.5.0",

    "jest-junit": "16.0.0",

    "prettier": "2.8.8",

    "source-map-support": "0.5.21",

    "sqlite": "5.0.1",

    "supertest": "6.3.3",

    "ts-jest": "29.1.0",

    "ts-loader": "9.4.3",

    "ts-node": "10.9.1",

    "tsconfig-paths": "4.2.0",

    "typescript": "5.1.3"

  },

  "jest": {

    "moduleFileExtensions": [

      "js",

      "json",

      "ts"

    ],

    "rootDir": "src",

    "testRegex": ".*\\.spec\\.ts$",

    "transform": {

      "^.+\\.(t|j)s$": "ts-jest"

    },

    "collectCoverageFrom": [

      "**/*.(t|j)s"

    ],

    "coverageDirectory": "../coverage",

    "testEnvironment": "node",

    "moduleNameMapper": {

      "^@/(.*)$": "<rootDir>/$1"

    }

  }

}

  

  

//example.env

SERVER_AUTH=

SERVER_PORT=

SERVER_PREFIX=

NODE_ENV=

SERVER_JWT_SECRET=

SERVER_JWT_EXP=

NX_AGGRID_KEY=

RMAP_DATABASE_USERNAME=

RMAP_DATABASE_PASSWORD=

RMAP_DATABASE_HOST=

RMAP_DATABASE_NAME=

RMAP_DATABASE_PORT=

RMAP_DATABASE_SCHEMA=

UDS_DATABASE_HOST=

UDS_DATABASE_PORT=

UDS_DATABASE_NAME=

UDS_DATABASE_SCHEMA=

UDS_DATABASE_USERNAME=

UDS_DATABASE_PASSWORD=

ROUTER_LEGACY_API=

SUDIR_API=

ROUTER_CONNECTION_COUNT=

ROUTER_ENDPOINT=

SPANNER_HOST=

SPANNER_AUTH=

SUDIR_ROLES_HOST=

SPANNER_PSWD=

SPANNER_USER=

  

  

//tsconfig.json

{

  "compilerOptions": {

    "module": "commonjs",

    "declaration": true,

    "removeComments": true,

    "emitDecoratorMetadata": true,

    "experimentalDecorators": true,

    "allowSyntheticDefaultImports": true,

    "target": "ES2021",

    "sourceMap": true,

    "outDir": "./dist",

    "baseUrl": "./",

    "incremental": true,

    "skipLibCheck": true,

    "strictNullChecks": false,

    "noImplicitAny": false,

    "strictBindCallApply": false,

    "forceConsistentCasingInFileNames": false,

    "noFallthroughCasesInSwitch": false,

    "paths": {

      "@modules": ["src/modules"],

      "@modules/*": ["src/modules/*"],

      "@middlewares":["src/common/middlewares"],

      "@common/types":["src/common/types/types/"],

      "@common/interfaces":["src/common/types/interfaces"],

      "@common/enums":["src/common/types/enums"],

      "@middlewares/*":["src/common/middlewares/*"],

      "@guards":["src/common/guards"],

      "@guards/*":["src/common/guards/*"],

      "@filters":["src/common/filters"],

      "@filters/*":["src/common/filters/*"],

      "@decorators ":["src/decorators"],

      "@decorators/* ":["src/decorators/*"],

      "@migrations":["src/migrations"],

      "@migrations/*":["src/migrations/*"],

      "@environments":["src/environments"],

      "@environments/*":["src/environments/*"],

      "@configs/*":["src/configs/*"],

      "@common/providers":["src/common/providers/"],

      "@common/providers/*":["src/common/providers/*"],

    }

  }

}

  

  

// Конфигурирование HttpModule (axios) в модуле

  

  

import { ConfigModule, ConfigService } from '@nestjs/config';

import { SudirRolesService } from './sudirRoles.service';

import { Module } from '@nestjs/common';

import { HttpModule, HttpModuleOptions } from '@nestjs/axios';

  

  

@Module({

    imports: [

        ConfigModule,

        HttpModule.registerAsync({

            useFactory: (configService: ConfigService): HttpModuleOptions => {

                return {

                    baseURL: configService.get<string>('SUDIR_ROLES_HOST'),

                    maxRedirects: 0,

                    headers: {},

                    timeout: 150000,

                }

            },

            inject: [ConfigService]

        }),

    ],

    providers: [ SudirRolesService ],

    exports:  [ SudirRolesService ],

})

export class SudirRolesModule { }

  

  

// Использование сконфигурированного HttpModule

import { BadRequestException, ForbiddenException, Injectable, NotFoundException, UnauthorizedException } from '@nestjs/common';

import { ConfigService } from '@nestjs/config';

import { HttpService } from '@nestjs/axios';

import { AxiosInstance, AxiosResponse } from 'axios';

import { LoggerService } from '../logger/logger.service';

import { GetUserInfoInterface, SudirRolesUserInterface } from './sudirRoles.type';

  

  

@Injectable()

export class SudirRolesService {

  

  

    constructor(

        private readonly configService: ConfigService,

        private readonly httpService: HttpService,

        private readonly loggerService: LoggerService,

    ){

        this.axios = httpService.axiosRef;

    }

  

  

    private axios: AxiosInstance;

  

  

    async getUserInfo(pdi: string): Promise<any> {

  

  

        this.loggerService.warn(`getUserInfo(): Запуск метода сервиса`)

        try {

            const url = `/InternalServices/userByCreds`;

            this.loggerService.warn(`getUserInfo(): запрос в API ${url}  pdi пользователя ${pdi}`);

            const response = await this.axios.post(url, {creds: pdi});

  

  

            return response?.data;

        } catch (error) {

            if(error?.response) this.loggerService.error(`getUserInfo(): Cтатус ${error.response.status} не удалось получить пользователя ${error.response}`);

            else if(error?.message) this.loggerService.error(`getUserInfo(): Нe удалось получить пользователя ${error.message}`)

            else this.loggerService.error(`getUserInfo(): Нe удалось получить пользователя ${error}`)

        }

    }

}