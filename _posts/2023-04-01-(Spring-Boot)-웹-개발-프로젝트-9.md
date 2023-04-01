---
layout: post
title: (Spring Boot) 웹 개발 프로젝트 - 9
subtitle: 프로필 구현 - 1 (등록)
categories: Project
tags: [Spring, Java, Web, ]
---

## 프로필 기능 설명
프로필 기능은 이 프로젝트에서 핵심적인 내용이라고 말할 수 있다. SNS의 핵심이 자신의 프로필을 생성하고, 자신만의 스토리를 다른사람에게 공개하는 것인것 처럼 이 프로젝트에서도 커플들이 본인들의 사랑이야기, 데이트 내용 등을 프로필을 통해 다른 사람들에게 공개하는 것이 핵심이다. 이러한 프로필 기능은 크게 포스트 등록, 프로필 화면 설계가 있다. 이번 포스팅에선 포스트 등록 관련해서 이야기하도록 하겠다. 

## 포스트 등록 설계
### 데이터베이스 설계
 등록에 있어서 가장먼저 해야할 일은 포스트관련 테이블을 생성해야한다. 포스트에 있어야할 내용으로는 포스트 ID, 남자친구 아이디, 여자친구 아이디, 이미지 정보, 내용, 해시태그이다. 이때, 이미지 정보의 경우는 이미지가 여러개일 수 있고, 원본 이미지 파일 이름과 저장되는 이미지 파일 이름이 모두 필요하다. 따라서, List형태로 이미지 여러개를 저장할 수 있고, 두가지 형태의 파일 이름을 멤버변수로 갖는 UploadFile이라는 클래스형식으로 저장하여야 한다.
이 조건들을 충족하기 위해서는 테이블이 2개가 필요하다. 포스트 내용을 포함하고있는 POST테이블과 POST테이블의 images컬럼의 내용을 포함하고 있는 UPLOAD_FILE 테이블을 모두 생성하여 join하여야한다.
```profile/Post.java
@Data
@Entity
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotNull
    private String boyFriend;

    @NotNull
    private String girlFriend;

    @OneToMany(cascade = CascadeType.ALL)
    @JoinColumn(name = "post_id")
    private List<UploadFile> images = new ArrayList<>();

    private String content;

    private String hashTag;

}
```
먼저, POST 테이블 관련 클래스를 생성하였다. 이미지를 제외한 변수들은 String으로 선언하였고, 이미지의 경우 여러개가 필요하고 이미지 당 두가지 형태의 파일이름이 필요하기 때문에 List<UploadFile>형으로 선언하였다 그리고 UPLOAD_FILE테이블과 조인할 것이기 때문에 외래키를 post_id라는 이름으로 설정하였다. 
  
```profile/UploadFile.java
@Data
@Entity
public class UploadFile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String uploadFileName;

    private String storeFileName;

    public UploadFile(String uploadFileName, String storeFileName) {
        this.uploadFileName = uploadFileName;
        this.storeFileName = storeFileName;
    }

    public UploadFile() {

    }
}
```  
그리고 두가지 형태의 파일 이름을 저장하기 위한 UploadFile클래스를 생성해주었다. Post클래스에서 images를 List<UploadFile>형으로 선언하였기 때문에 자동으로 스프링을 통해서 Post클래스와 join한다.
<br>
  
### 화면 설계
먼저 포스트 등록화면을 설계할 것이다.
```templates/page/profilePage_add.html
<!DOCTYPE html>
<html th:replace="~{layout/common_layout :: layout(~{::title}, ~{::section})}">
<head>
  <title>러브쉐어</title>
</head>
<body>
<section class="text-center">
  <form th:action method="post" enctype="multipart/form-data">
    <fieldset>
      <legend>게시물 등록</legend>
      <div class="form-group">
        <label for="images" class="form-label mt-4" style="float: left">이미지</label>
        <input class="form-control" type="file" multiple="multiple" id="images" name="images">
      </div>
      <div class="form-group">
      <label for="content" class="form-label mt-4" style="float: left">내용</label>
      <textarea class="form-control" id="content" name="content" rows="3"></textarea>
      </div>
      <div class="form-group">
        <label class="col-form-label mt-4" for="hashTag" style="float: left">해시태그</label>
        <input type="text" class="form-control" placeholder="해시태그를 추가해보세요" id="hashTag" name="hashTag">
      </div><br>
      <button type="submit" class="btn btn-primary">등록</button>
    </fieldset>
  </form>
</section>
</body>
</html>
```
세션을 통해서 남자친구, 여자친구 이름 정보가 현재 존재하기 때문에 포스트를 등록하기 위해서 사용자로부터 요구해야할 정보는 이미지파일, 포스트 내용, 해시태그이다. 여기서 해시태그는 나중에 검색기능을 사용할 때 쓰이는 포스트정보이다. 
받아올 정보를 <input>태그를 통해서 받아온다. 여기서 중요한 점은 images의 경우 여려개의 이미지 파일을 받을 수 있도록 하여야하기 때문에 속성으로 'type="file" multiple="multiple"'을 추가해야한다. 여기서 file은 사용자로부터 file을 받을 수 있도록 하고, multiple을 여러개의 파일을 한번에 받을 수 있도록 하는 속성이다. 
  
등록을 하면, 포스트를 볼 수 있도록 해야하기 때문에 포스트화면도 설계할 것이다.
```templates/page/profilePage_post.html
<!DOCTYPE html>
<html th:replace="~{layout/common_layout :: layout(~{::title}, ~{::section})}">
<head>
  <title>러브쉐어</title>
</head>
<body>
<section class="text-center">
  <span th:text="${post.boyFriend}" style="font-size: 20px"></span>
  <span style="font-size: 20px">&hearts;</span>
  <span th:text="${post.girlFriend}"  style="font-size: 20px"></span><br><br>
  <div id="carouselControls" class="carousel slide" data-bs-ride="carousel" style="width:360px;height:480px;margin: auto;">
    <div class="carousel-inner">
      <div class="carousel-item" th:each="image, i : ${post.images}"
           th:classappend="${i.index == 0} ? 'active'">
        <img th:src="|/images/${image.getStoreFileName()}|" width="360" height="480"/>
      </div>
    </div>
    <button class="carousel-control-prev" type="button" data-bs-target="#carouselControls" data-bs-slide="prev">
      <span class="carousel-control-prev-icon" aria-hidden="true"></span>
      <span class="visually-hidden">Previous</span>
    </button>
    <button class="carousel-control-next" type="button" data-bs-target="#carouselControls" data-bs-slide="next">
      <span class="carousel-control-next-icon" aria-hidden="true"></span>
      <span class="visually-hidden">Next</span>
    </button>
  </div><br><br>
  <div class="card border-primary mb-3" style="width:360px;margin: auto;">
    <div class="card-body">
      <span class="card-text" th:text="${post.content}" style="font-size: 18px">글내용</span><br>
      <span class="card-text" th:text="${post.hashTag}" style="font-size: 15px">해시태크</span>
    </div>
  </div>
</section>
</body>
</html>
```
포스트 화면은 먼저제목으로 커플의 아이디가 보이고, 여러개의 이미지를 carousel을 통해서 넘겨볼수 있도록 하였고, 내용과 해시태그를 이미지 밑에 보이도록 하였다. 이미지의 경우 이미지의 개수만큼 반복문의 반복을 통해 모두 볼 수 있도록 화면을 구성하였다. 또한, 버튼을 통해 이미지를 넘길 수 있도록 하였다. 이러한 carousel기능은 기존에 생성하였던 common_layout.html에 "<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.0/dist/js/bootstrap.bundle.min.js"></script>"를 추가함으로서 구현되도록 하였다.
<br>
  
## 포스트 등록 등록
### 파일 저장 클래스 생성
사용자로부터 받아온 이미지파일에 대해 저장소에 저장될 이미지 파일 이름을 생성하고, 저장소에 저장할 수 있도록 하는 클래스가 필요하다. 

```profile/FileStore.class
@Component
public class FileStore {

    @Value("${file.dir}")
    private String fileDir;

    public String getFullPath(String filename) {
        return fileDir + filename;
    }

    public List<UploadFile> storeFiles(List<MultipartFile> multipartFiles) throws IOException {
        List<UploadFile> storeFileResult = new ArrayList<>();
        for (MultipartFile multipartFile : multipartFiles) {
            if (!multipartFile.isEmpty()) {
                storeFileResult.add(storeFile(multipartFile));
            }
        }
        return storeFileResult;
    }

    public UploadFile storeFile(MultipartFile multipartFile) throws IOException {
        if (multipartFile.isEmpty()) {
            return null;
        }

        String originalFilename = multipartFile.getOriginalFilename();
        String storeFileName = createStoreFileName(originalFilename);
        multipartFile.transferTo(new File(getFullPath(storeFileName)));
        return new UploadFile(originalFilename, storeFileName);
    }

    private String createStoreFileName(String originalFilename) {
        String ext = extractExt(originalFilename);
        String uuid = UUID.randomUUID().toString();
        return uuid + "." + ext;
    }

    private String extractExt(String originalFilename) {
        int pos = originalFilename.lastIndexOf(".");
        return originalFilename.substring(pos + 1);
    }
}
```  
  
클래스가 매우 복잡하게 보일 수 있으니 메소드별로 하나씩 설명하도록 하겠다. 
 - @Value("${file.dir}") : 파일이 저장될 위치를 나타낸다.
 - extractExt() : 파일의 형식을 문자열로 뽑아내는 메소드
 - createStoreFileName() : uuid를 생성하여 파일의 형식 문자열과 합쳐 storeFileName을 생성하는 메소드
 - storeFile() : storeFileName으로이미지 파일을 File객체를 통해 파일이 저장될 위치에 저장하는 메소드, UploadFile객체형으로 반
 - storeFiles() : storeFile() 메소드를 사용하여 여러 파일을 저장하는 메소드, 저장된 이미지파일을 List<UploadFile>형으로 반환

전체적으로 보면 여러 이미지 파일들을 받아와 각각의 이미지 파일의 uuid형식인 storeFileName을 생성하여 설정해놓은 저장소에 File객체를  storeFileName으로 저장되도록한다.
  
### 포스트 저장
Post클래스에서 멤버변수 images는 원본이미지 이름과, 파일을 저장할 때, 생성되는 저장용이름 정보만 담고 있기 때문에 사용자로부터 포스트 내용을 전달해주는 profilePage_add.html과 직접적으로 연결될 수 없다. 따라서, 사용자로부터 받는 포스트내용을 저장할 수 있는 도메인인 PostForm을 따로 생성해주어야한다.
```profile/PostForm.class
@Data
public class PostForm {

    private List<MultipartFile> images;
    private String content;
    private String hashTag;
}
```
 profilePage_add.html에서 받아오는 이미지 파일은 MultiparFile형식이기 때문에 PostForm의 멤버변수 images를 List<MultipartFile>형으로 선언해주었다.
