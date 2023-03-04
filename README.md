# MRSK

MRSK, sıfır kesinti süresiyle Docker kullanan çıplak donanımdan bulut VM'lerine kadar her yerde web uygulamalarını dağıtır. Yeni uygulama kapsayıcısını başlatırken ve eskisini durdururken istekleri tutmak için dinamik ters proxy Traefik kullanır. Komutları yürütmek için SSHKit kullanarak birden çok ana bilgisayarda sorunsuz çalışır. Rails uygulamaları için oluşturulmuştur, ancak Docker ile kapsayıcıya alınabilen herhangi bir web uygulamasıyla çalışır.

Watch the screencast: https://www.youtube.com/watch?v=LL1cV2FXZ5I

## Kurulum

"gem install mrsk" ile MRSK'yi global olarak kurun. Ardından, uygulama dizininizin içinde "mrsk init" komutunu çalıştırın (veya bir bin/mrsk binstub istediğiniz Rails uygulamalarında "mrsk init --bundle"). Şimdi yeni `config/deploy.yml` dosyasını düzenleyin. Bu kadar basit görünebilir:

```yaml
service: hey
image: 37s/hey
servers:
  - 192.168.0.1
  - 192.168.0.2
registry:
  username: registry-user-name
  password:
    - MRSK_REGISTRY_PASSWORD
env:
  secret:
    - RAILS_MASTER_KEY
```

Ardından, kayıt defteri parolanızı "MRSK_REGISTRY_PASSWORD" (ve bir Rails uygulamasıyla üretim için "RAILS_MASTER_KEY") olarak eklemek için ".env" dosyanızı düzenleyin.

Artık sunuculara dağıtmaya hazırsınız:
```
mrsk dağıtma
```

  This will:

1. SSH üzerinden sunuculara bağlanın (varsayılan olarak kök kullanarak, kimliği ssh anahtarınız tarafından doğrulanır)
2. Docker'ı eksik olabilecek herhangi bir sunucuya kurun (apt-get kullanarak)
3. Kayıt defterinde hem yerel hem de uzaktan oturum açın
4. Uygulamanın kök dizinindeki standart Dockerfile'ı kullanarak görüntüyü oluşturun.
5. Görüntüyü kayıt defterine itin.
6. Görüntüyü kayıt defterinden sunuculara çekin.
7. Traefik'in 80 numaralı bağlantı noktasında çalıştığından ve trafiği kabul ettiğinden emin olun.
8. Uygulamanızın "GET /up" için "200 OK" ile yanıt verdiğinden emin olun.
9. Geçerli git sürümü karması ile eşleşen uygulama sürümüyle yeni bir kapsayıcı başlatın.
10. Uygulamanın önceki sürümünü çalıştıran eski kapsayıcıyı durdurun.
11. Sunucuların dolmamasını sağlamak için kullanılmayan görüntüleri ve durdurulan kapsayıcıları budayın.

İşte! Artık tüm sunucular uygulamayı 80 numaralı bağlantı noktasında sunuyor. Yalnızca tek bir sunucu çalıştırıyorsanız, gitmeye hazırsınız. Birden fazla sunucu çalıştırıyorsanız, önlerine bir yük dengeleyici koymanız gerekir.

## Vizyon

Geçtiğimiz on yılda+, web uygulamalarının dağıtımını kolaylaştıran ticari tekliflerde bir patlama oldu. Heroku, görünüşte sonsuza kadar rekabetin önünde kalan inanılmaz bir teklifle başladı. Bu günlerde Fly.io ve Render gibi mükemmel alternatiflerimiz var. Barındırılan Kubernetes, AWS, GCP, Digital Ocean ve başka yerlerde de işleri kolaylaştırıyor. Ancak bunların tümü, bulutta yüksek ücret karşılığında bilgisayar kiralamanızı sağlayan tekliflerdir. Kendi donanımınız üzerinde çalıştırmak istiyorsanız, hatta gelecekte bunu yapmak için net bir geçiş yolunuz varsa, bu ticari platformlara ne kadar kilitlendiğinizi dikkatlice düşünmeniz gerekir. Faturalar işinizi bir bütün olarak yutmadan önce tercih edin!

MRSK, bu ticari tekliflerin öncülük ettiği ergonomideki ilerlemeyi web uygulamalarını her yerde dağıtmaya getirmeyi amaçlıyor. İster Digital Ocean, Hetzner, OVH, vb. gibi yönetilen hizmet işaretlemesi olmadan düşük maliyetli bulut seçenekleri, ister kendi ortak konumlu çıplak metaliniz olsun. MRSK için hepsi aynı. Yapılandırma dosyasına, eklenen bir SSH anahtarının ötesinde hiçbir hazırlık görmemiş vanilya Ubuntu sunucularına sahip IP adreslerinin bir listesini verin ve kelimenin tam anlamıyla birkaç dakika içinde çalışmaya başlayacaksınız.

Bu yaklaşım size muazzam taşınabilirlik sağlar. Web uygulamanızın bu şekilde kolayca birkaç bulutta konuşlandırılmasını sağlayabilirsiniz. Ya da kendi donanımınızla taban hattını satın alabilir, ardından daha fazla kapasite elde etmek için büyük bir sezonluk artıştan önce bir buluta dağıtabilirsiniz. Takımlama açısından tek bir sağlayıcıya bağlı kalmadığınızda, pek çok ilgi çekici seçenek mevcuttur.

Nihayetinde MRSK, herhangi bir ticari teklife bağlı olmayan açık kaynak araçları kullanarak üretime geçmenin karmaşıklığını azaltmayı amaçlıyor. Sıfır değil, dikkat et. Temel Linux veya Docker hala zorsa tam olarak yönetilen bir hizmetle muhtemelen daha iyi durumdasınızdır, ancak bu kavramlara aşina olur olmaz, MRSK ile devam etmeye hazır olacaksınız.

## Neden sadece Capistrano, Kubernetes veya Docker Swarm'ı çalıştırmıyorsunuz?

MRSK, sunucuları önceden dikkatli bir şekilde hazırlamaya gerek kalmadan temel olarak Konteynerler için Capistrano'dur. Sunucuların doğru Ruby sürümüne veya ihtiyacınız olan diğer bağımlılıklara sahip olduğundan emin olmanıza gerek yok. Bunların hepsi artık Docker görüntüsünde yaşıyor. Yepyeni bir Ubuntu (veya her neyse) sunucusunu önyükleyebilir, onu MRSK'deki sunucular listesine ekleyebilir ve Docker ile otomatik olarak sağlanır ve hemen çalıştırılır. Docker'ın katman önbelleğe alma özelliği, sunucuda daha az uğraşarak konuşlandırmaları da hızlandırır. MRSK için oluşturulan görüntüler, CI veya daha sonra iç gözlem için kullanılabilir.

Kubernetes bir canavardır. Kendi donanımınız üzerinde kendiniz çalıştırmak, kalbin zayıflığı için değildir. Şeffaf bir şekilde [Render gibi](https://thenewstack.io/render-cloud-deployment-with-less-engineering/) veya açıkça AWS/GCP'de başka birinin platformunda çalıştırmak istiyorsanız bu iyi bir seçenektir, ancak bulut ile kendi donanımınız arasında hareket etme, hatta ikisini karıştırma özgürlüğü istiyorsanız, MRSK çok daha basittir. Olan biten her şeyi görebilirsiniz, sadece çağrılan temel Docker komutları.

Docker Swarm, Kubernetes'ten çok daha basittir, ancak yine de durum mutabakatını kullanan aynı bildirim modeli üzerine inşa edilmiştir. MRSK, Capistrano gibi zorunlu komutlar etrafında kasıtlı olarak tasarlanmıştır.

Sonuç olarak, web uygulamalarını dağıtmanın sayısız yolu vardır, ancak bu, [37signals](https://37signals.com)'da [HEY](https://www.hey.com) getirmek için kullandığımız araç setidir. ) [buluttan eve](https://world.hey.com/dhh/why-we-re-leaving-the-cloud-654b47e0) modern konteynerleştirme araçlarının avantajlarını kaybetmeden.

## Configuration

### Gerekli ortam değişkenlerini yüklemek için .env dosyasını kullanma

MRSK, uygulama kökünde bulunan ".env" dosyasında ayarlanan ortam değişkenlerini otomatik olarak yüklemek için [dotenv](https://github.com/bkeepers/dotenv) kullanır. Bu dosya, MRSK_REGISTRY_PASSWORD gibi değişkenleri veya veritabanı şifrelerini ayarlamak için kullanılabilir. Ancak bu nedenle .env dosyalarının Git'e teslim edilmediğinden veya Dockerfile'ınıza dahil edilmediğinden emin olmalısınız! Biçim, şuna benzer bir anahtar/değer çiftidir:

```bash
MRSK_REGISTRY_PASSWORD=pw
DB_PASSWORD=secret123
```

### Oluşturulmuş bir .env dosyası kullanma

#### 1password as a secret store

1Password gibi merkezi bir gizli depo kullanıyorsanız, sırları arayan bir şablon olarak `.env.erb` oluşturabilirsiniz. .env.erb dosyası örneği:

```erb
<% if (session_token = `op signin --account my-one-password-account --raw`.strip) != "" %># Generated by mrsk envify
GITHUB_TOKEN=<%= `gh config get -h github.com oauth_token`.strip %>
MRSK_REGISTRY_PASSWORD=<%= `op read "op://Vault/Docker Hub/password" -n --session  #{session_token}` %>
RAILS_MASTER_KEY=<%= `op read "op://Vault/My App/RAILS_MASTER_SECRET" -n --session #{session_token}` %>
MYSQL_ROOT_PASSWORD=<%= `op read "op://Vault/My App/MYSQL_ROOT_PASSWORD" -n --session #{session_token}` %>
<% else raise ArgumentError, "Session token missing" end %>
```

Bu şablon güvenle git'te kontrol edilebilir. Ardından, uygulamayı dağıtan herkes, uygulamayı ilk kez kurduklarında "mrsk envify"ı çalıştırabilir veya doğru ".env" dosyasını almak için parolaları değiştirebilir.

Farklı hedefler için ayrı env değişkenlerine ihtiyacınız varsa, bunları "mrsk envify -d staging" ile çalıştırıldığında ".env.staging" oluşturacak olan şablon için ".env.destination.erb" ile ayarlayabilirsiniz.

#### gizli depo olarak bitwarden

Bitwarden gibi açık kaynaklı bir gizli depo kullanıyorsanız, sırları arayan bir şablon olarak `.env.erb` oluşturabilirsiniz.

`SOME_SECRET`i bitwarden kasasında güvenli bir notta saklayabilirsiniz.
```
$ bw list items --search SOME_SECRET | jq
? Master password: [hidden]

[
  {
    "object": "item",
    "id": "123123123-1232-4224-222f-234234234234",
    "organizationId": null,
    "folderId": null,
    "type": 2,
    "reprompt": 0,
    "name": "SOME_SECRET",
    "notes": "yyy",
    "favorite": false,
    "secureNote": {
      "type": 0
    },
    "collectionIds": [],
    "revisionDate": "2023-02-28T23:54:47.868Z",
    "creationDate": "2022-11-07T03:16:05.828Z",
    "deletedDate": null
  }
]
```

ve "SOME_SECRET"in "id"ini yukarıdaki "json"dan çıkarın ve aşağıdaki "erb"de kullanın.

Örnek `.env.erb` dosyası:

```erb
<% if (session_token=`bw unlock --raw`.strip) != "" %># Generated by mrsk envify
SOME_SECRET=<%= `bw get notes 123123123-1232-4224-222f-234234234234 --session #{session_token}` %>
<% else raise ArgumentError, "session_token token missing" end %>
```

Ardından, uygulamayı dağıtan herkes "mrsk envify" komutunu çalıştırabilir ve mrsk, ".env" dosyasını oluşturur.

### Docker Hub dışında başka bir kayıt defteri kullanma

Varsayılan kayıt defteri Docker Hub'dır, ancak bunu "kayıt defteri/sunucu" kullanarak değiştirebilirsiniz:

```yaml
registry:
  server: registry.digitalocean.com
  username: registry-user-name
  password: <%= ENV.fetch("MRSK_REGISTRY_PASSWORD") %>
```

### Kökten farklı bir SSH kullanıcısı kullanma

Varsayılan SSH kullanıcısı root'tur, ancak bunu "ssh/user" kullanarak değiştirebilirsiniz:

```yaml
ssh:
  user: app
```

### Proxy SSH ana bilgisayarı kullanma

Sunucuya bir proxy ana bilgisayarı aracılığıyla bağlanmanız gerekiyorsa, "ssh/proxy" kullanabilirsiniz:

```yaml
ssh:
  proxy: "192.168.0.1" # defaults to root as the user
```

Veya belirli bir kullanıcıyla:

```yaml
ssh:
  proxy: "app@192.168.0.1"
```
### env değişkenlerini kullanma

"env" kullanarak env değişkenlerini uygulama kapsayıcılarına enjekte edebilirsiniz:

```yaml
env:
  DATABASE_URL: mysql2://db1/hey_production/
  REDIS_URL: redis://redis1:6379/1
```

### Using secret env variables

If you have env variables that are secret, you can divide the `env` block into `clear` and `secret`:

```yaml
env:
  clear:
    DATABASE_URL: mysql2://db1/hey_production/
    REDIS_URL: redis://redis1:6379/1
  secret:
    - DATABASE_PASSWORD
    - REDIS_PASSWORD
```

Gizli ortam değişkenlerinin listesi, çalışma zamanında yerel makinenizden genişletilecektir. Bu nedenle, gizli bir "DATABASE_PASSWORD" referansı, MRSK çalıştıran makinede "ENV["DATABASE_PASSWORD"]" ifadesini arayacaktır. Tıpkı yapı sırlarında olduğu gibi.

Başvurulan gizli ENV'ler eksikse, yapılandırma bir "KeyError" istisnasıyla durdurulur.

Not: Bir ENV'yi gizli olarak işaretlemek şu anda yalnızca MRSK çıktısındaki değerini çıkarır. ENV, çalışma zamanında şeffaf olarak konteynere enjekte edilir.

### Birimleri kullanma

"Birimler"i kullanarak uygulama kapsayıcılarına özel birimler ekleyebilirsiniz:

```yaml
volumes:
  - "/local/path:/container/path"
```

### Sunucular için farklı roller kullanma

Uygulamanız, varsayılan web çalışmasının ötesindeki işleri veya diğer rolleri çalıştırmak için ayrı ana bilgisayarlar kullanıyorsa, bu ana bilgisayarları aşağıdaki gibi yeni bir giriş noktası komutuyla özel bir rolde belirtebilirsiniz:

```yaml
servers:
  web:
    - 192.168.0.1
    - 192.168.0.2
  job:
    hosts:
      - 192.168.0.3
      - 192.168.0.4
    cmd: bin/jobs
```

Not: Traefik varsayılan olarak yalnızca "web" rolündeki sunuculara (ve herhangi bir rol tanımlanmamışsa tüm sunuculara) yüklenecek ve çalışacaktır. Traefik'e "web" dışındaki rollerde sahip makinelerde ihtiyacınız varsa, "traefik: true" ekleyin:

```yaml
servers:
  web:
    - 192.168.0.1
    - 192.168.0.2
  web2:
    traefik: true
    hosts:
      - 192.168.0.3
      - 192.168.0.4
```

### Kapsayıcı etiketleri kullanma
Başlatılmakta olan kapsayıcılara etiketler ayarlayarak varsayılan Traefik kurallarını özelleştirebilirsiniz:

```
labels:
  traefik.http.routers.hey.rule: Host(\`app.hey.com\`)
```

Not: Kuralın doğru bir şekilde iletildiğinden ve Bash tarafından komut ikamesi olarak değerlendirilmediğinden emin olmak için kaçan ters işaretler gereklidir!

Bu, aynı Traefik örneğini ve bağlantı noktasını paylaşan aynı sunucuda birden fazla uygulama çalıştırmanıza olanak tanır.See https://doc.traefik.io/traefik/routing/routers/#rule for a full list of available routing rules.

The labels can also be applied on a per-role basis:

```yaml
servers:
  web:
    - 192.168.0.1
    - 192.168.0.2
  job:
    hosts:
      - 192.168.0.3
      - 192.168.0.4
    cmd: bin/jobs
    labels:
      my-label: "50"
```

### Yerel çoklu arşiv için uzak oluşturucuyu kullanma

ARM64 (Apple Silicon gibi) üzerinde geliştirme yapıyorsanız, ancak AMD64 (x86 64 bit) üzerinde dağıtım yapmak istiyorsanız, çoklu mimari görüntüleri kullanabilirsiniz. Varsayılan olarak MRSK, bunu QEMU emülasyonu aracılığıyla yapan yerel bir buildx yapılandırması kuracaktır. Ancak bu, özellikle ilk derlemede oldukça yavaş olabilir.

ARM64 bölümünü yerel olarak yerel olarak oluştururken görüntünün AMD64 bölümünü yerel olarak oluşturmak için uzak bir AMD64 ana bilgisayarı kullanarak bu işlemi hızlandırmak istiyorsanız, bunu oluşturucu seçeneklerini kullanarak yapabilirsiniz:

```yaml
builder:
  local:
    arch: arm64
    host: unix:///Users/<%= `whoami`.strip %>/.docker/run/docker.sock
  remote:
    arch: amd64
    host: ssh://root@192.168.0.1
```

Not: Oluşturucu olarak kullanılan uzak ana bilgisayarda Docker'ın çalışıyor olması gerekir. Bu örnek, yalnızca aynı kayıt defterini ve kimlik bilgilerini kullanan derlemeler için paylaşılmalıdır.

### Tek kemer için uzak oluşturucuyu kullanma

ARM64 (Apple Silicon gibi) üzerinde geliştirme yapıyorsanız, AMD64 (x86 64 bit) üzerinde dağıtım yapmak istiyorsanız, ancak görüntüyü yerel olarak (veya diğer ARM64 ana bilgisayarlarında) çalıştırmanız gerekmiyorsa, bir uzak oluşturucu yapılandırabilirsiniz. sadece AMD64'ü hedefler. Yerel olarak inşa edilecek bir şey olmadığı için bu, çoklu kemer ile inşa etmekten biraz daha hızlıdır.

```yaml
builder:
  remote:
    arch: amd64
    host: ssh://root@192.168.0.1
```

### Çoklu arşiv gerekli olmadığında yerel oluşturucuyu kullanma

Dağıtmakta olduğunuz mimariyle aynı mimari üzerinde geliştirme yapıyorsanız, hem çok kemerli hem de uzak yapıdan vazgeçerek yapıyı hızlandırabilirsiniz:

```yaml
builder:
  multiarch: false
```

TMRSK'yi dağıtım sunucularıyla mimariyi paylaşan bir CI sunucusundan çalıştırıyorsanız, bu da iyi bir seçenektir.

### Yeni görüntüler için yapı sırlarını kullanma

Bazı görüntülerin, özel mücevher havuzlarına erişim sağlamak için oluşturma sırasında GITHUB_TOKEN gibi bir sırrın iletilmesi gerekir. Bu, sırrı ENV'de bulundurarak ve ardından oluşturucu yapılandırmasında ona başvurarak yapılabilir:

```yaml
builder:
  secrets:
    - GITHUB_TOKEN
```

Bu derleme sırrına daha sonra Dockerfile'da başvurulabilir:

```dockerfile
# Copy Gemfiles
COPY Gemfile Gemfile.lock ./

# Install dependencies, including private repositories via access token (then remove bundle cache with exposed GITHUB_TOKEN)
RUN --mount=type=secret,id=GITHUB_TOKEN \
  BUNDLE_GITHUB__COM=x-access-token:$(cat /run/secrets/GITHUB_TOKEN) \
  bundle install && \
  rm -rf /usr/local/bundle/cache
```

### Traefik için komut bağımsız değişkenlerini kullanma

traefik komut satırını özelleştirebilirsiniz:

```yaml
traefik:
  args:
    accesslog: true
    accesslog.format: json
```
Bu, `--accesslog=true accesslog.format=json` ile traefik kapsayıcısını başlatacaktır.

### Yeni görüntüler için yapı bağımsız değişkenlerini yapılandırma

Gizli olmayan derleme bağımsız değişkenleri de yapılandırılabilir:

```yaml
builder:
  args:
    RUBY_VERSION: 3.2.0
```

Bu derleme bağımsız değişkeni daha sonra Dockerfile'da kullanılabilir:
```
# Private repositories need an access token during the build
ARG RUBY_VERSION
FROM ruby:$RUBY_VERSION-slim as base
```

### Veritabanı, önbellek, arama hizmetleri için aksesuarları kullanma

Aksesuar hizmetlerinizi de MRSK üzerinden yönetebilirsiniz. Hizmetler, herkese açık görüntüler oluşturacak ve dağıttığınızda otomatik olarak güncellenmeyecek:

```yaml
accessories:
  mysql:
    image: mysql:5.7
    host: 1.1.1.3
    port: 3306
    env:
      clear:
        MYSQL_ROOT_HOST: '%'
      secret:
        - MYSQL_ROOT_PASSWORD
    volumes:
      - /var/lib/mysql:/var/lib/mysql
  redis:
    image: redis:latest
    host: 1.1.1.4
    port: "36379:6379"
    volumes:
      - /var/lib/redis:/data
```

Şimdi MySQL sunucusunu 1.1.1.3'te başlatmak için `mrsk aksesuarı start mysql` komutunu çalıştırın. Mümkün olan tüm komutlar için mrsk aksesuarına bakın.

### Cron'u Kullanma

Cron işlerinizi çalıştırmak için belirli bir kapsayıcı kullanabilirsiniz:

```yaml
servers:
  cron:
    hosts:
      - 192.168.0.1
    cmd:
      bash -c "cat config/crontab | crontab - && cron -f"
```
Bu, Cron ayarlarının "config/crontab" içinde saklandığını varsayar.

### Denetim yayınlarını kullanma

Dağıtım, geri alma vb. denetimlerini bir sohbet odasına veya başka bir yere yayınlamak isterseniz, denetim satırından ilk bağımsız değişken olarak geçirilecek bir bin dosyasının yolu ile "audit_broadcast_cmd" ayarını yapılandırabilirsiniz:

```yaml
audit_broadcast_cmd:
  bin/audit_broadcast
```

Yayın komutu şöyle görünür:
```bash
#!/usr/bin/env bash
curl -q -d content="[My App] ${1}" https://3.basecamp.com/XXXXX/integrations/XXXXX/buckets/XXXXX/chats/XXXXX/lines
```

Bu, Basecamp'ta önceden yapılandırılmış bir sohbet robotuna aşağıdaki gibi bir satır gönderir:

```
[My App] [dhh] Rolled back to version d264c4e92470ad1bd18590f04466787262f605de
```

### Özel durum denetimi yolu veya bağlantı noktası kullanma

MRSK varsayılan olarak uygulamanızın durumunu 3000 numaralı bağlantı noktasında "/up" yeniden kontrol eder. Her ikisini de "healthcheck" ayarıyla uyarlayabilirsiniz:

```yaml
healthcheck:
  path: /healthz
  port: 4000
```

Bu, uygulamanızın `/healthz` karşısında sağlık kontrolü için traefik etiketiyle yapılandırılmasını ve MRSK'nin gerçekleştirdiği dağıtım öncesi sağlık kontrolünün 4000 numaralı bağlantı noktasında aynı yola karşı yapılmasını sağlar.

## Komutlar

### Sunucularda komut çalıştırma

Sunucularda tek seferlik komutları çalıştırabilirsiniz:

```bash
# Komutu tüm sunucularda çalıştırır
mrsk app exec 'ruby -v'
App Host: 192.168.0.1
ruby 3.1.3p185 (2022-11-24 revision 1a6b16756e) [x86_64-linux]

App Host: 192.168.0.2
ruby 3.1.3p185 (2022-11-24 revision 1a6b16756e) [x86_64-linux]

# Komutu birincil sunucuda çalıştırır
mrsk app exec --primary 'cat .ruby-version'
App Host: 192.168.0.1
3.1.3

# Runs Rails command on all servers
mrsk app exec 'bin/rails about'
App Host: 192.168.0.1
About your application's environment
Rails version             7.1.0.alpha
Ruby version              ruby 3.1.3p185 (2022-11-24 revision 1a6b16756e) [x86_64-linux]
RubyGems version          3.3.26
Rack version              2.2.5
Middleware                ActionDispatch::HostAuthorization, Rack::Sendfile, ActionDispatch::Static, ActionDispatch::Executor, Rack::Runtime, Rack::MethodOverride, ActionDispatch::RequestId, ActionDispatch::RemoteIp, Rails::Rack::Logger, ActionDispatch::ShowExceptions, ActionDispatch::DebugExceptions, ActionDispatch::Callbacks, ActionDispatch::Cookies, ActionDispatch::Session::CookieStore, ActionDispatch::Flash, ActionDispatch::ContentSecurityPolicy::Middleware, ActionDispatch::PermissionsPolicy::Middleware, Rack::Head, Rack::ConditionalGet, Rack::ETag, Rack::TempfileReaper
Application root          /rails
Environment               production
Database adapter          sqlite3
Database schema version   20221231233303

App Host: 192.168.0.2
About your application's environment
Rails version             7.1.0.alpha
Ruby version              ruby 3.1.3p185 (2022-11-24 revision 1a6b16756e) [x86_64-linux]
RubyGems version          3.3.26
Rack version              2.2.5
Middleware                ActionDispatch::HostAuthorization, Rack::Sendfile, ActionDispatch::Static, ActionDispatch::Executor, Rack::Runtime, Rack::MethodOverride, ActionDispatch::RequestId, ActionDispatch::RemoteIp, Rails::Rack::Logger, ActionDispatch::ShowExceptions, ActionDispatch::DebugExceptions, ActionDispatch::Callbacks, ActionDispatch::Cookies, ActionDispatch::Session::CookieStore, ActionDispatch::Flash, ActionDispatch::ContentSecurityPolicy::Middleware, ActionDispatch::PermissionsPolicy::Middleware, Rack::Head, Rack::ConditionalGet, Rack::ETag, Rack::TempfileReaper
Application root          /rails
Environment               production
Database adapter          sqlite3
Database schema version   20221231233303

# Rails runner'ı birincil sunucuda çalıştırın
mrsk app exec -p 'bin/rails runner "puts Rails.application.config.time_zone"'
UTC
```

### SSH üzerinden etkileşimli komutlar çalıştırma

Bir sunucuda Rails konsolu veya bash oturumu gibi etkileşimli komutlar çalıştırabilirsiniz (varsayılan birincildir, diğerine bağlanmak için --hosts'u kullanın):

```bash
# Starts a bash session in a new container made from the most recent app image
mrsk app exec -i bash

# Starts a bash session in the currently running container for the app
mrsk app exec -i --reuse bash

# Starts a Rails console in a new container made from the most recent app image
mrsk app exec -i 'bin/rails console'
```


### Kapsayıcıların durumunu göstermek için çalışan ayrıntılar

`mrsk detaylarını` çalıştırarak sunucularınızın durumunu görebilirsiniz:

```
Traefik Host: 192.168.0.1
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                               NAMES
6195b2a28c81   traefik   "/entrypoint.sh --pr…"   30 minutes ago   Up 19 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp   traefik

Traefik Host: 192.168.0.2
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                               NAMES
de14a335d152   traefik   "/entrypoint.sh --pr…"   30 minutes ago   Up 19 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp   traefik

App Host: 192.168.0.1
CONTAINER ID   IMAGE                                                                         COMMAND                  CREATED          STATUS          PORTS      NAMES
badb1aa51db3   registry.digitalocean.com/user/app:6ef8a6a84c525b123c5245345a8483f86d05a123   "/rails/bin/docker-e…"   13 minutes ago   Up 13 minutes   3000/tcp   chat-6ef8a6a84c525b123c5245345a8483f86d05a123

App Host: 192.168.0.2
CONTAINER ID   IMAGE                                                                         COMMAND                  CREATED          STATUS          PORTS      NAMES
1d3c91ed1f55   registry.digitalocean.com/user/app:6ef8a6a84c525b123c5245345a8483f86d05a123   "/rails/bin/docker-e…"   13 minutes ago   Up 13 minutes   3000/tcp   chat-6ef8a6a84c525b123c5245345a8483f86d05a123
```

Ayrıca, "mrsk uygulama ayrıntıları" ile yalnızca uygulama kapsayıcıları için veya "mrsk traefik ayrıntıları" ile yalnızca Traefik için bilgileri görebilirsiniz.

### Kötü bir dağıtımı düzeltmek için geri alma çalıştırılıyor

If you've discovered a bad deploy, you can quickly rollback by reactivating the old, paused container image. You can see what old containers are available for rollback by running `mrsk app containers`. It'll give you a presentation similar to `mrsk app details`, but include all the old containers as well. Showing something like this:

```
App Host: 192.168.0.1
CONTAINER ID   IMAGE                                                                         COMMAND                  CREATED          STATUS                      PORTS      NAMES
1d3c91ed1f51   registry.digitalocean.com/user/app:6ef8a6a84c525b123c5245345a8483f86d05a123   "/rails/bin/docker-e…"   19 minutes ago   Up 19 minutes               3000/tcp   chat-6ef8a6a84c525b123c5245345a8483f86d05a123
539f26b28369   registry.digitalocean.com/user/app:e5d9d7c2b898289dfbc5f7f1334140d984eedae4   "/rails/bin/docker-e…"   31 minutes ago   Exited (1) 27 minutes ago              chat-e5d9d7c2b898289dfbc5f7f1334140d984eedae4

App Host: 192.168.0.2
CONTAINER ID   IMAGE                                                                         COMMAND                  CREATED          STATUS                      PORTS      NAMES
badb1aa51db4   registry.digitalocean.com/user/app:6ef8a6a84c525b123c5245345a8483f86d05a123   "/rails/bin/docker-e…"   19 minutes ago   Up 19 minutes               3000/tcp   chat-6ef8a6a84c525b123c5245345a8483f86d05a123
6f170d1172ae   registry.digitalocean.com/user/app:e5d9d7c2b898289dfbc5f7f1334140d984eedae4   "/rails/bin/docker-e…"   31 minutes ago   Exited (1) 27 minutes ago              chat-e5d9d7c2b898289dfbc5f7f1334140d984eedae4
```

Yukarıdaki örnekten, "e5d9d7c2b898289dfbc5f7f1334140d984eedae4" dosyasının son sürüm olduğunu görebiliriz, dolayısıyla bir geri alma hedefi olarak kullanılabilir. Bu geri dönüşü mrsk rollback e5d9d7c2b898289dfbc5f7f1334140d984eedae4' komutunu çalıştırarak gerçekleştirebiliriz. Bu, "6ef8a6a84c525b123c5245345a8483f86d05a123" ifadesini durdurur ve ardından "e5d9d7c2b898289dfbc5f7f1334140d984eedae4" ifadesini başlatır. Eski konteyner hala mevcut olduğu için bu çok hızlıdır. Kayıt defterinden indirilecek bir şey yok.

`mrsk konuşlandırması`nı çalıştırdığınızda, varsayılan olarak eski kapların 3 gün sonra budandığını unutmayın.

### Sunucuları temizlemek için kaldırma işlemi çalıştırılıyor

Traefik, kapsayıcılar, resimler ve kayıt oturumu dahil olmak üzere tüm uygulamayı kaldırmak isterseniz, mrsk remove komutunu çalıştırabilirsiniz. Bu, sunucuları temiz bırakacaktır.

## Geliştirme aşaması

Bu bir beta yazılımıdır. Komutlar yine de hareket edebilir. Ancak [37signals](https://37signals.com) adresinde canlı yayındayız.

## Lisans

MRSK, [MIT Lisansı](https://opensource.org/licenses/MIT) altında yayınlanmıştır.
