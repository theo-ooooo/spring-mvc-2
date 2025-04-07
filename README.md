# 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술 (Jar, Thymeleaf)
## 🧱 MVC 프레임워크 만들기

스프링이 내부에서 어떻게 MVC 구조를 처리하는지 이해하기 위해, 직접 간단한 MVC 프레임워크를 구현해보며 구조와 역할을 학습했습니다.

### ✅ 프론트 컨트롤러 도입

요청을 단일 진입점(FrontController)으로 받아, URL에 따라 컨트롤러를 분기 처리하는 구조를 직접 구현했습니다.

```java
@WebServlet(name = "frontController", urlPatterns = "/front-controller/v1/*")
public class FrontControllerServletV1 extends HttpServlet {
    private Map<String, ControllerV1> controllerMap = new HashMap<>();

    public FrontControllerServletV1() {
        controllerMap.put("/front-controller/v1/members/new-form", new MemberFormControllerV1());
        controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV1());
        controllerMap.put("/front-controller/v1/members", new MemberListControllerV1());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String requestURI = request.getRequestURI();
        ControllerV1 controller = controllerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }
        controller.process(request, response);
    }
}
```

---

### ✅ 컨트롤러 인터페이스로 역할 분리

모든 컨트롤러가 동일한 인터페이스를 구현하도록 하여 확장 가능성을 확보했습니다.

```java
public interface ControllerV1 {
    void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```

---

### ✅ 뷰 분리 및 중복 제거

반복되는 JSP 경로 생성을 `MyView`라는 뷰 객체로 추상화했습니다.

```java
public class MyView {
    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

---

### ✅ Model, View 분리와 도입 (V3, V4)

이후 단계에서는 다음과 같은 구조까지 확장했습니다:

- 핸들러가 `ModelView` 객체를 반환
- `ViewResolver`를 통해 논리 뷰 이름을 실제 경로로 변환
- 핸들러 어댑터를 도입해 다양한 컨트롤러 버전 처리

```java
public class ModelView {
    private String viewName;
    private Map<String, Object> model = new HashMap<>();

    // ... getter/setter 생략
}
```

---

### 📁 구조 예시

```
front-controller
├── v1  (서블릿 직접 호출, HttpServletRequest/Response 그대로 사용)
├── v2  (MyView로 뷰 렌더링 분리)
├── v3  (ModelView 도입)
├── v4  (파라미터 간소화, 컨트롤러 유연화)
├── v5  (핸들러 어댑터 도입)
```

---

이 과정을 통해 스프링 MVC의 구조와 철학을 직접 체감할 수 있었고, DispatcherServlet의 동작 원리를 명확하게 이해하게 되었습니다.
