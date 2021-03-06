---
id: 893
title: 'PictOgr &#8211; cebulka + moje pierwsze DDD.'
date: 2017-04-26T10:06:06+00:00
author: gocom
layout: post
guid: http://godev.gemustudio.com/?p=893
permalink: /2017/04/26/pictogr-cebulka-pierwsze-ddd/
image: /assets/images/2017/04/480-Zwiebel-zwiebel-Fotolia-28405313-c-vadim-yerofeyev.jpg
categories:
  - Daj Się Poznać 2017
  - PictOgr
tags:
  - CQRS
  - dajsiepoznac2017
  - DDD
  - DSP2017
  - Onion
  - PictOgr
---
<div id="dslc-theme-content">
  <div id="dslc-theme-content-inner">
    <p style="text-align: justify;">
      Udało się<a href="http://godev.gemustudio.com/assets/images/2017/04/onion.png"><img class="alignright wp-image-905 size-full" src="http://godev.gemustudio.com/assets/images/2017/04/onion.png" alt="Clean Architecture" width="194" height="206" /></a> ugotować cebulkę, projekt wygląda znacznie lepiej aniżeli wcześniej. I dodatkowo ma większe możliwości.
    </p>
    
    <p style="text-align: justify;">
      Stworzyłem też moje pierwsze <strong>DDD (Domain-Driven Design)</strong>, ostatnio zachorowałem w tym kierunku (tak jak CQRS i Onion), i pragnę zgłębiac temat&#8230;
    </p>
    
    <p style="text-align: justify;">
      Zmiana architektury na tak wczesnym etapie projektu nie była zbyt bolesna. Tym bardziej, iż CQRS został wyodrębniony wcześniej.
    </p>
    
    <p style="text-align: justify;">
      <strong>Jest to moje pierwsze praktyczne zetknięcie z Clean Architecture (lamer).</strong>
    </p>
    
    <p>
      <strong>PictOgr</strong> obecnie składa się z 7 projektów tak jak to widać na pierwszym zrzucie.
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <h1 style="background: #ffff9c; padding: 5pt; text-align: left;">
      Zmiany
    </h1>
    
    <p style="text-align: justify;">
      <a href="http://godev.gemustudio.com/assets/images/2017/04/pictogr_infrastructure.png"><img class="alignleft wp-image-906 size-full" src="http://godev.gemustudio.com/assets/images/2017/04/pictogr_infrastructure.png" alt="Onion Architecture" width="174" height="254" /></a>Wydzieliłem warstwę domeny w projekcie Core.
    </p>
    
    <p style="text-align: justify;">
      Infrastructure zawiera wykorzystane usług, <strong>CQRS</strong>, implementacje repozytoriuów, <strong>Autofac</strong>, <strong>DTO</strong>, <strong>AutoMapper</strong>, i inne potrzebne elementy, bardziej szczegółowy wykaz na drugim zrzucie.
    </p>
    
    <p style="text-align: justify;">
      GUI aplikacji znajdować się będzie w projekcie <strong>MVVM</strong>, czyli XAMLe, <strong>ViewModele</strong> oraz moduły dla <strong>Autofaca</strong>.
    </p>
    
    <p style="text-align: justify;">
      Dodałem też projekt, w którym znajdą się testy integracyjne o nazwie E2E.
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <p style="text-align: center;">
      <strong>Zmiana architektury niesie ze sobą kilka zmian dotyczących mechanizmów działania aplikacji!</strong>
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <h3 style="background: #ffff9c; padding: 5pt; text-align: left;">
      Użycie eventa do zamykania okien
    </h3>
    
    <p style="text-align: justify;">
      Na pierwszy ogień, klasa implementująca (<strong>IEvent</strong>) zdarzenie zamykania aplikacji.
    </p>
    
    <pre class="lang:c# decode:true" title="Zdarzenie zamykania aplikacji.">using CQRS.Event;

namespace PictOgr.Infrastructure.Events
{
	public class ExitApplicationEvent : IEvent
	{
	    public ExitApplicationEvent(int exitCode)
	    {
	        
	    }
	}
}</pre>
    
    <p>
      &nbsp;
    </p>
    
    <p style="text-align: justify;">
      Powyższa klasa jest wykorzystywana przez powiązanego Handlera o nazwie <strong>ExitApplicationEventHandler</strong>.
    </p>
    
    <p style="text-align: justify;">
      Jej celem  jest wywołanie przekazanego <strong>delegata</strong> do zamykania apikacji w metodzie <strong>Handle</strong>. To spowoduje zamknięcie okienka.
    </p>
    
    <pre class="lang:c# decode:true" title="Handler zdarzenia zamykania aplikacji.">using System;
using CQRS.Event;

namespace PictOgr.Infrastructure.Events
{
    public class ExitApplicationEventHandler : IEventHandler&lt;ExitApplicationEvent&gt;
    {
        private readonly Action action;

        public ExitApplicationEventHandler(Action action)
        {
            this.action = action;
        }

        public void Handle(ExitApplicationEvent @event)
        {
            action();
        }
    }
}
</pre>
    
    <p>
      &nbsp;
    </p>
    
    <p style="text-align: justify;">
      Rejestrowanie handlera i przekazanie delegato do zamknięcia okienka.
    </p>
    
    <pre class="lang:c# decode:true" title="Rejestrowanie handlera do zdarzenia zamykania aplikacji.">eventBus.Register(new ExitApplicationEventHandler(() =&gt;
{
	Close();
}));</pre>
    
    <p>
      &nbsp;
    </p>
    
    <p style="text-align: justify;">
      Pozostaje jedynie w odpowiedniej komendzie <strong>ExitApplicationHandler</strong> wykonać publikacje zdarzenia <strong>ExitApplicationEvent </strong>do szyny zdarzeń.
    </p>
    
    <p style="text-align: justify;">
      Efektem jest zamknięcie wszystkich okienek w których zarejestrowany jest handler <strong>ExitApplicationEventHandler</strong> (kod wyżej).
    </p>
    
    <pre class="lang:c# decode:true " title="Publikowanie zdarzenia na szynę w handlerze comendy zamykania aplikacji.">public void Handle(ExitApplication command)
{
    _logger.Info("Exit application.");
    _eventBus.Publish(new ExitApplicationEvent(command.ExitCode));
    System.Environment.Exit(command.ExitCode);
}</pre>
    
    <p>
      &nbsp;
    </p>
    
    <h3 style="background: #ffff9c; padding: 5pt; text-align: left;">
      Wykorzystanie usług w zapytaniach CQRSa
    </h3>
    
    <p style="text-align: justify;">
      Do przekazywania danych pomiędzy aplikacją, a modelem wykorzystana będzie odrębna klasa określana jako <strong>DTO (Data Transfer Object)</strong>.
    </p>
    
    <p style="text-align: justify;">
      Dzięki takiemu odseparowaniu aplikacja nic nie wie o modelu domeny jaki zaimplementujemy w aplikacji.
    </p>
    
    <pre class="lang:c# decode:true" title="Klasa informacji o aplikacji, DTO - Data Transfer Object, do przekazania informacji od modelu domenowego.">namespace PictOgr.Infrastructure.DTO
{
	public class ApplicationInformationDto
	{
		public string Version { get; set; }
	}
}</pre>
    
    <p>
      Dane pomiędzy modelem domeny a klasą DTO muszą być przekazane, i można to zrobić pod górkę poprostu przepisaując właściwości z wykorzystaniem klasy pośredniczącej (np. jakiegoś Adaptera), lub zywkorzytaniem biblioteczki <strong><a href="http://automapper.org/">AutoMapper</a></strong>.
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <p>
      Poniżej znajduje się konfiguracja AutoMappera dla klas <strong>ApplicationInformation</strong> <=> <strong>ApplicationInformationDto</strong>.
    </p>
    
    <pre class="lang:c# decode:true " title="Mapowanie obiektu domeny na obiekt DTO.">using AutoMapper;
using PictOgr.Core.Domain;
using PictOgr.Infrastructure.DTO;

namespace PictOgr.Infrastructure.Mappers
{
	public static class AutoMapperConfig
	{
		public static IMapper Initialize() =&gt; new MapperConfiguration(config =&gt;
		{
			config.CreateMap&lt;ApplicationInformation, ApplicationInformationDto&gt;();
			config.CreateMap&lt;ApplicationInformationDto, ApplicationInformation&gt;();

		}).CreateMapper();
	}
}
</pre>
    
    <p style="text-align: justify;">
      Konfiguracje ustawiamy dwu kierunkowo oznacza to iż będzie można mapować dane w obu kierunkach:
    </p>
    
    <p style="text-align: justify;">
      <strong>ApplicationInformation</strong> = <strong>ApplicationInformationDto</strong>
    </p>
    
    <p style="text-align: justify;">
      <strong>ApplicationInformationDto</strong> = <strong>ApplicationInformation</strong>
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <p style="text-align: justify;">
      Do pobierania danych z domeny użyjemy tym razem usługi, do jej implementacji wykorzytsamy interfejsik <strong>IApplicaitonService</strong>, zawierający definicję metody pobierania informacji o aplikacji.
    </p>
    
    <pre class="lang:c# decode:true" title="Interfejsik dla usług aplikacji.">using PictOgr.Infrastructure.DTO;

namespace PictOgr.Infrastructure.Services.ApplicationService
{
	public interface IApplicationService
	{
		ApplicationInformationDto GetApplicationInformation();
	}
}</pre>
    
    <p>
      &nbsp;
    </p>
    
    <p style="text-align: justify;">
      Interfejs <strong>IApplicaitonService</strong>, jest zaimplementowany przez poniższą klasę <strong>ApplicationService</strong>.
    </p>
    
    <pre class="lang:c# decode:true" title="Usługa aplikacji implementująca swój interfejsik.">using AutoMapper;
using PictOgr.Core.Domain;
using PictOgr.Core.Repositories;
using PictOgr.Infrastructure.DTO;

namespace PictOgr.Infrastructure.Services.ApplicationService
{
	public class ApplicationService : IApplicationService
	{
		private readonly IApplicationInformationRepository _applicationInformationRepository;
		private readonly IMapper _mapper;

		public ApplicationService(
			IApplicationInformationRepository applicationInformationRepository,
			IMapper mapper)
		{
			_applicationInformationRepository = applicationInformationRepository;
			_mapper = mapper;
		}

		public ApplicationInformationDto GetApplicationInformation()
		{
			var applicationInformation = _applicationInformationRepository.GetApplicationInformation();

			return _mapper.Map&lt;ApplicationInformation, ApplicationInformationDto&gt;(applicationInformation);
		}
	}
}
</pre>
    
    <p style="text-align: justify;">
      W klasie usługi po pobraniu danych z repozytorium odbywa się mapowanie:
    </p>
    
    <p style="text-align: justify;">
      <strong><em>return _mapper.Map<ApplicationInformation, ApplicationInformationDto>(applicationInformation);</em></strong>
    </p>
    
    <p style="text-align: justify;">
      Efektem jest przeniesienie danych z obiektu domenowego do obiektu DTO.
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <h3 style="background: #ffff9c; padding: 5pt; text-align: left;">
      Użycie modelu domeny do pobrania informacji o aplikacji
    </h3>
    
    <p style="text-align: justify;">
      Pierwszy obiekt w moim modelu domeny. To klasa z informacjami o aplikacji.
    </p>
    
    <p style="text-align: justify;">
      Bardzo banalna, różni się w zasadzie od klasy z nią powiązanej (<strong>DTO</strong>), wykorzystaniem konstruktora i możliwością ustawienia właściwości <strong>Version</strong> jedynie właśnie z tego konstruktora (lub metod klasy).
    </p>
    
    <pre class="lang:c# decode:true" title="Obiekt domeny zawierający informacje o aplikacji.">namespace PictOgr.Core.Domain
{
	public class ApplicationInformation
	{
		public ApplicationInformation(string version)
		{
			Version = version;
		}

		public string Version { get; private set; }
	}
}</pre>
    
    <p>
      &nbsp;
    </p>
    
    <p style="text-align: justify;">
      Jest tutaj też interfejsik z <strong>repozytorium</strong> pobierania informacji o aplikacji, owe repozytorium będzie implementowane dopiero w warstiwe wyżej (<strong>Infrastrukture</strong>). Repozytorium jest ściśle związane z modelem <strong>ApplicationInformation</strong>.
    </p>
    
    <pre class="lang:c# decode:true" title="Interfejsik repozytorium pobierania informacji o aplikacji.">using PictOgr.Core.Domain;

namespace PictOgr.Core.Repositories
{
	public interface IApplicationInformationRepository
	{
		ApplicationInformation GetApplicationInformation();
	}
}</pre>
    
    <p>
      &nbsp;
    </p>
    
    <p style="text-align: justify;">
      Jedna metoda pobierania informacji jest implementowana w klasie repozytorium <strong>ApplicationInformationRepository</strong>, implementuje ona interfejs z modelu domeny <strong>IApplicaitonInformationRepository</strong>, i dostarcza informacji o aplikacji.
    </p>
    
    <pre class="lang:c# decode:true " title="Implementacja repozytorium pobierania danych o aplikacji (warstwa infrastruktury).">using System.Reflection;
using PictOgr.Core.Domain;
using PictOgr.Core.Repositories;

namespace PictOgr.Infrastructure.Repositories
{
	public class ApplicationInformationRepository: IApplicationInformationRepository
	{
		public ApplicationInformation GetApplicationInformation()
		{
			var version = Assembly.GetExecutingAssembly().GetName().Version;
			return new ApplicationInformation($"{version.Major}.{version.Minor}.{version.Build}");
		}
	}
}</pre>
    
    <p style="text-align: justify;">
      Na chwilę obecną jest to tylko wersja plikacji, jednak z czasem może ulec rozbudowie o więcej ciekawych infomracji.
    </p>
    
    <p style="text-align: center;">
      <strong>To tyle z dużych zmian w projekcie, myślę iż teraz pujdzie już znacznie lepiej (dla oka = GUI).</strong>
    </p>
    
    <h3 style="background: #ffff9c; padding: 5pt; text-align: left;">
      Zakończenie
    </h3>
    
    <p style="text-align: justify;">
      Tak wiem jest to prosty przykład, zapewne wogule nie powinno się robić w ten sposób. Jednak się uczę, i taki przykład dostarcza mi dużo doświadczenia.
    </p>
    
    <p style="text-align: justify;">
      Dlatego też postanowiłem zrobić to w ten sposób, być może ktoś dzięki temu wpisowi zrozumie coś wiecej&#8230;
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <h3 style="text-align: center;">
      <strong>Dziękuję za wytrwałość i zachęcam do komentowania.</strong>
    </h3>
    
    <p>
      &nbsp;
    </p>
    
    <div>
      <div style="text-align: center;">
        <strong>Jest to post przygotowany na potrzeby konkursu &#8222;Daj Się Poznać 2017&#8221; organizowanym przez <a href="http://devstyle.pl">Macieja Aniserowicza</a>.</strong>
      </div>
      
      <div style="text-align: center;">
         <a href="http://devstyle.pl/daj-sie-poznac/" target="_blank" rel="noopener noreferrer"><img class="wp-image-104 size-full alignright" title="Daj Się Poznać 2017" src="http://godev.gemustudio.com/assets/images/2017/02/dsp2017-3.png" alt="" width="68" height="154" /></a>
      </div>
      
      <div style="font-size: 10pt; padding: 10px;">
        <div>
          <strong>Projekt: </strong><a href="http://godev.gemustudio.com/2017/03/01/pictogr-pomysl/">PictOgr</a>.
        </div>
        
        <div>
          <strong>GitHub: </strong><a href="https://github.com/krzysztofowsiany/pictogr">github.com/krzysztofowsiany/pictogr</a>
        </div>
        
        <div>
          <strong>Blog: </strong><a href="http://godev.gemustudio.com">godev.gemustudio.com</a>
        </div>
        
        <div>
          <strong>Snapchat</strong>: <a href="https://www.snapchat.com/add/gocom7" target="_blank" rel="noopener noreferrer">gocom7</a>
        </div>
        
        <div>
          <strong>RSS: </strong><a href="http://godev.gemustudio.com/category/daj-sie-poznac-2017/feed">godev.gemustudio.com/category/daj-sie-poznac-2017/feed</a>
        </div>
        
        <div>
          <strong>Facebook:</strong><a href="https://www.facebook.com/PictOgr-1729700930654225/">www.facebook.com/PictOgr-1729700930654225/</a>
        </div>
        
        <div>
          <strong>Twitter: </strong><a href="https://twitter.com/gemu_gocom">twitter.com/gemu_gocom</a>
        </div>
      </div>
    </div>
    
    <!-- AddThis Advanced Settings generic via filter on the_content -->
    
    <!-- AddThis Share Buttons generic via filter on the_content -->
  </div>
</div>