---
title: "Spring Boot + GDAL 라이브러리"
author: hcbak
date: 2025-05-13 21:58:26 +0900
categories: [Backend, Spring Boot]
---

## 환경

OS|Windows 11

## GDAL

GDAL(Geospatial Data Abstraction Library)은 래스터 및 벡터 지리공간 데이터 형식을 위한 오픈 소스 변환 라이브러리라고 한다.

자세히 아는 것은 아니고, 구직 중에 채용 절차를 따르면서 사전 과제로 기능 구현을 요구 받아서 기록을 남기려고 한다.  
(추후 프로젝트 외부 라이브러리를 사용하는 경우에도 참고할 수 있을 것 같다.)

일반적인 TIFF 파일을 COG(Cloud Optimized GeoTIFF) 형식으로 변환하는 것을 예시로 작성한다.

## GDAL 라이브러리 설치

GDAL 라이브러리는 단순히 의존성 추가로 사용할 수 있는 것이 아니다. 프로젝트 외부 라이브러리를 불러오는 의존성을 추가하는데, 결국 환경 변수에 GDAL 라이브러리가 등록되어 있어야 정상적으로 불러올 수 있다. 그래서 바깥에 먼저 GDAL 라이브러리를 설치하고, 환경 변수를 잡아줘야 한다.

[GISInternals](https://www.gisinternals.com/){:target="_blank"}에서 설치 파일을 다운로드할 수 있다. [Latest Build Status](https://www.gisinternals.com/status.php){:target="_blank"}에 접속하여 **Release builds**의 최신 버전(현 시점: [release-1930-x64-gdal-3-10-0-mapserver-8-2-2](https://www.gisinternals.com/query.html?content=filelist&file=release-1930-x64-gdal-3-10-0-mapserver-8-2-2.zip){:target="_blank"})으로 들어가서 Windows 버전 core(현 시점: [gdal-3.10.0-1930-x64-core.msi](https://download.gisinternals.com/sdk/downloads/release-1930-x64-gdal-3-10-0-mapserver-8-2-2/gdal-3.10.0-1930-x64-core.msi){:target="_blank"})로 설치하면 된다.

설치 파일이 아니더라도 압축 파일을 받아서 환경 변수에 추가해도 되는데, 개인적으로 설치 파일이 있다면 가급적 설치 파일을 사용하는 편이 관리가 수월해서 설치 파일로 진행했다.

### 환경 변수 설정

설치 파일이 별도의 환경 변수 설정을 하지 않아서 직접 환경 변수를 설정해야 한다.

|      | Key              | Value                                               |
| ---- | ---------------- | --------------------------------------------------- |
| 생성 | GDAL_DATA        | C:\Program Files\GDAL\gdal-data                     |
| 생성 | GDAL_DRIVER_PATH | C:\Program Files\GDAL\gdalplugins                   |
| 생성 | PROJ_LIB         | C:\Program Files\GDAL\projlib                       |
| 추가 | Path             | C:\Program Files\GDAL<br>C:\Program Files\GDAL\java |

PowerShell을 관리자 권한으로 열고 아래 명령어를 입력해도 되고, 검색으로 **시스템 환경 변수 편집**을 찾아서 환경 변수를 설정해도 된다.

```powershell
[Environment]::SetEnvironmentVariable("GDAL_DATA", "C:\Program Files\GDAL\gdal-data", 2)
[Environment]::SetEnvironmentVariable("GDAL_DRIVER_PATH", "C:\Program Files\GDAL\gdalplugins", 2)
[Environment]::SetEnvironmentVariable("PROJ_LIB", "C:\Program Files\GDAL\projlib", 2)
[Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path", 2) + ";C:\Program Files\GDAL;C:\Program Files\GDAL\java", 2)
```

혹시라도 버전이 상승하며 경로가 다르게 잡힐 수 있으니 설치 폴더를 확인하여 경로를 잡아주면 된다.

## Spring Boot

### 의존성

프로젝트 외부의 프로그램을 사용하지만 프로젝트 내에서 각종 명령어를 사용해야 하기 때문에 의존성 추가를 해줘야 한다.  
(만약 하지 않는다면 환경 변수로 설정한 경로의 실행 파일에 대해서 exec 등으로 직접 명령어를 시스템으로 보내야 한다.)

/build.gradle

```gradle
// ...

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.gdal:gdal:3.10.1'
}

// ...
```

### 변환

#### 명령어

먼저 명령어로 실행하는 경우에 다음과 같은 명령어를 사용한다.

```powershell
# COG 변환
gdal_translate origin.tiff result.tiff -of COG

# BigTIFF
gdal_translate origin.tiff result.tiff -of COG -co BIGTIFF=YES

# 멀티 스레드
gdal_translate origin.tiff result.tiff -of COG -co NUM_THREADS=ALL_CPUS

# COG + BigTIFF + 멀티 스레드
gdal_translate origin.tiff result.tiff -of COG -co BIGTIFF=YES -co NUM_THREADS=ALL_CPUS
```

BigTIFF는 기존 32bit 오프셋의 한계인 4GiB 이상의 파일의 경우에 필요한 옵션이다. 멀티 스레드는 자연수로 옵션을 줄 수 있고, ALL_CPUS는 가용한 모든 자원을 사용하는 것을 의미한다.

#### 코드

먼저 GDAL 드라이버를 등록한다. 중복해서 등록할 필요성은 없을 것 같아서 서비스 시작과 함께 등록하도록 했다.

/src/main/java/project/ProjectApplication.java

```java
@SpringBootApplication
public class ProjectApplication {

	public static void main(String[] args) {
		gdal.AllRegister();
		SpringApplication.run(ProjectApplication.class, args);
	}

}
```

적당히 Controller와 Service로 파일을 받아와서 아래 Service로 넘겼다. GDAL을 대강 어떻게 사용하는지 느낌만 보면 된다.

/src/main/java/project/translate/serivce/GdalService.java

```java
@Service
public class GdalService {

    public String tiffToCog(String path) {
        // COG 파일 경로 설정
        String cogFile = new StringBuilder()
            .append(FilenameUtils.getPath(path))
            .append("cog\\")
            .append(FilenameUtils.getBaseName(path))
            .append(".tiff")
            .toString();

        // 원본 파일 로드
        Dataset dataset = gdal.Open(path);

        // 변환 옵션 설정
        Vector<String> optionsVector = new Vector<>();
        optionsVector.add("-of");
        optionsVector.add("COG");
        optionsVector.add("-co");
        optionsVector.add("BIGTIFF=YES");
        optionsVector.add("-co");
        optionsVector.add("NUM_THREADS=ALL_CPUS");
        TranslateOptions options = new TranslateOptions(optionsVector);
        
        // [COG 파일 경로, 원본 파일, 변환 옵션] → COG 파일
        Dataset cogDataset = gdal.Translate(cogFile, dataset, options);
        
        // Dataset 해제 및 원본 파일 삭제
        dataset.delete();
        cogDataset.delete();
        new File(path).delete();

        return cogFile;
    }
}
```

마지막에 Dataset.delete()를 하지 않으면 사용중인 파일로 인식하여 삭제가 불가능할 수 있다.

## 참고

- GDAL documentation
  - Raster drivers
    - [COG -- Cloud Optimized GeoTIFF generator](https://gdal.org/en/stable/drivers/raster/cog.html){:target="_blank"}
  - Java
    - Package org.gdal.gdal
      - [Class gdal](https://gdal.org/en/stable/java/org/gdal/gdal/gdal.html){:target="_blank"}
- GISInternals
  - [Build Status](https://www.gisinternals.com/status.php){:target="_blank"}
    - Release builds
- GitHub
  - OSGeo/gdal
    - Issues
      - Gdal Java Binding not finding gdalalljni.dll: Can't find dependent libraries in version 204
        - [issuecomment-1314871316](https://github.com/OSGeo/gdal/issues/890#issuecomment-1314871316){:target="_blank"}
- Microsoft
  - Learn
    - .NET
      - API browser
        - Environment
          - [GetEnvironmentVariable(String, EnvironmentVariableTarget)](https://learn.microsoft.com/en-us/dotnet/api/system.environment.getenvironmentvariable?view=net-9.0#system-environment-getenvironmentvariable(system-string-system-environmentvariabletarget)){:target="_blank"}
          - [SetEnvironmentVariable(String, String, EnvironmentVariableTarget)](https://learn.microsoft.com/en-us/dotnet/api/system.environment.setenvironmentvariable?view=net-9.0#system-environment-setenvironmentvariable(system-string-system-string-system-environmentvariabletarget)){:target="_blank"}
        - [EnvironmentVariableTarget Enum](https://learn.microsoft.com/en-us/dotnet/api/system.environmentvariabletarget?view=net-9.0){:target="_blank"}
