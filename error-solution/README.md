# 에러 및 해결책 정리

## 1. 타임리프

#### 1) org.thymeleaf.exceptions.TemplateInputException: Error resolving template 에러 발생

* 로컬 개발 환경에서는 정상적으로 동작 하였지만, 리눅스 서버에 배포하면 컨트롤러에서 뷰 이름을 리턴할 때, 맨 앞에 `/`가 있어서 에러가 발생했다.

     ```java
    @GetMapping("/board/{id}/modify")
    public String updateBoardForm(@CurrentAccount Account account, @PathVariable Long id, Model model) {
        …
    
        return “/board/modify"
    }
     ```
  
* 그래서 다음과 같이 맨 앞에 있는 `/`를 제거하여 해결하였다.

     ```java
    @GetMapping("/board/{id}/modify")
    public String updateBoardForm(@CurrentAccount Account account, @PathVariable Long id, Model model) {
        …
    
        return “board/modify"
    }
     ```

