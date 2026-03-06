apache.poi ile yapılmış bir bavkend ön tarafta da react ile yapılmış bir ekran var. burada ppt tasarımı yapılıyor. ben bu tasarıma drawing toolsnile çizilen şekiller yine highlighting isaretmeler vs eklemeliyim 
10 web sayfasını oku
Merhaba! React ile yaptığınız ön yüz tasarımına çizim araçları (drawing tools) ve vurgulama (highlighting) özellikleri eklemek için harika bir konuya değindiniz. Mevcut mimarinizde (React ön yüz + Apache POI arka plan) bu özellikleri hayata geçirmek için iki temel yaklaşımınız var.

Öncelikle, ihtiyacınız olan temel kavramları ve bu kavramların hem ön yüzde (React) hem de arka yüzde (Apache POI) nasıl karşılık bulduğunu bir tabloda özetleyelim:

Özellik	React (Ön Yüz)	Apache POI (Arka Yüz)
Şekil Çizme	Fabric.js veya React Design Editor gibi kütüphanelerle kullanıcının ekrana şekil (kare, daire, çizgi vb.) çizmesi sağlanır .	HSLF (.ppt) için HSLFSlide.createPicture() veya SLGraphics sınıfı ; XSLF (.pptx) için XSLFSlide.createShape() metotları kullanılır .
Metin Vurgulama	Kullanıcının seçtiği metin, butonlar veya kısayollar aracılığıyla işaretlenir. Seçilen metin ve vurgu rengi backend'e iletilir.	XSLF (.pptx) için XSLFTextRun.setHighlightColor() metodu doğrudan kullanılır . HSLF (.ppt) için doğrudan bir metot yoktur; bunun yerine metnin arka planına renkli bir şekil yerleştirme gibi yöntemler kullanılır .
Bu tablodan yola çıkarak, geliştirme sürecinizi iki ana aşamada planlayabiliriz.

🎨 React ile Ön Yüzde Çizim ve Vurgulama Deneyimi
Ön yüzde, kullanıcılarınızın PowerPoint'teki gibi özgürce şekil çizebileceği ve metinleri vurgulayabileceği bir arayüz oluşturmalısınız. Bunun için sıfırdan kod yazmak yerine, mevcut güçlü kütüphanelerden faydalanabilirsiniz.

Çizim Araçları İçin: Fabric.js en popüler çözümlerden biridir. React ile uyumlu çalışan bu kütüphane, tuval üzerinde şekiller oluşturma, sürükleme, yeniden boyutlandırma ve silme gibi tüm interaktif özellikleri hazır olarak sunar. Daha kapsamlı bir çözüm arıyorsanız, React Design Editor gibi Fabric.js üzerine inşa edilmiş projeler de mevcuttur .

Metin Vurgulama İçin: Kullanıcıların metin seçip bir butona tıklayarak vurgulama yapmasını sağlayabilirsiniz. Bu işlem sonucunda, vurgulanan metin parçası ("bu metin vurgulanacak") ve vurgu rengi ("sarı") backend'inize gönderilecek veriler olacaktır.

☕️ Apache POI ile Arka Yüzde PPT'yi Oluşturma
React'ten gelen verilerle (şeklin türü, boyutu, konumu; vurgulanacak metin ve rengi) arka yüzünüzde Apache POI kullanarak PowerPoint dosyasını oluşturacaksınız. Burada dikkat etmeniz gereken en önemli nokta, .ppt (Office 97-2003) ve .pptx (Office 2007 ve sonrası) dosya formatları için Apache POI'nin farklı modüller sunmasıdır.

Yeni Nesil (.pptx) Dosyalar İçin: XSLF* (XML Slide Layout) sınıflarını kullanmalısınız. Bu, daha modern ve önerilen yoldur.

Şekil Eklemek: XSLFSlide nesnesi üzerinden createAutoShape() gibi metotları kullanarak kare, daire, ok gibi hazır şekiller ekleyebilirsiniz .

Metin Vurgulamak: Bir metin kutusu (XSLFTextBox) ekleyip, içindeki bir metin parçasını (XSLFTextRun) vurgulamak oldukça kolaydır. Aşağıdaki örnek, bir .pptx dosyasında metni nasıl vurgulayacağınızı gösterir:

java
import org.apache.poi.xslf.usermodel.*;
import java.awt.*;
import java.io.FileOutputStream;

public class HighlightInPPTX {
    public static void main(String[] args) throws Exception {
        try (XMLSlideShow ppt = new XMLSlideShow()) {
            XSLFSlide slide = ppt.createSlide();
            // Bir metin kutusu oluştur
            XSLFTextBox textBox = slide.createTextBox();
            textBox.setAnchor(new Rectangle(50, 50, 300, 50));
            
            // Metin kutusuna bir paragraf ve metin ekle
            XSLFTextParagraph paragraph = textBox.addNewTextParagraph();
            XSLFTextRun run = paragraph.addNewTextRun();
            run.setText("Bu metin vurgulanacak.");
            
            // **Metni vurgula (sarı renk ile)**
            run.setHighlightColor(Color.YELLOW);

            try (FileOutputStream out = new FileOutputStream("vurgulu_sunum.pptx")) {
                ppt.write(out);
            }
        }
    }
}
Eski Nesil (.ppt) Dosyalar İçin: HSLF* (Horrible Slide Layout) sınıflarını kullanmalısınız.

Şekil Eklemek: HSLFSlide.createPicture() ile resim ekleyebilir veya daha karmaşık çizimler için SLGraphics sınıfını kullanarak bir GroupShape üzerine Java AWT komutlarıyla çizim yapabilirsiniz . Bu yöntem, serbest el çizimleri için idealdir.

Metin Vurgulamak: .pptx dosyalarındaki kadar doğrudan bir destek yoktur. Bir geçici çözüm olarak, vurgulamak istediğiniz metnin arkasına, metinle aynı boyutlarda ve konumda, yarı saydam renkli bir dikdörtgen (HSLFAutoShape) yerleştirebilirsiniz. Bu, görsel olarak vurgulama efekti yaratacaktır .

🚀 Mimari Akış
Kullanıcı React uygulamanızda bir şekil çizer veya bir metni vurgular.

React uygulamanız, bu eylemi bir JSON nesnesine dönüştürür. Örneğin: { "eylem": "sekilEkle", "tip": "dikdortgen", "x": 100, "y": 150, "genislik": 200, "yukseklik": 100 } veya { "eylem": "metinVurgula", "metin": "örnek metin", "renk": "sari" }.

Bu JSON verisini bir API çağrısı ile Java backend'inize iletirsiniz.

Backend'inizde bir servis, gelen JSON'u parse eder. Eğer hedef format .pptx ise XSLF sınıflarıyla, .ppt ise HSLF sınıflarıyla gerekli işlemleri yaparak sunuyu oluşturur veya günceller.

Oluşturulan dosyayı kullanıcıya indirirsiniz.

💡 Ek Öneriler
Karmaşık Çizimler: Çok karmaşık, serbest el çizimleri veya özel vektörel grafikler için arka yüzde SLGraphics sınıfını kullanmanız gerekebilir. Bu, size bir Java Graphics2D nesnesi gibi davranarak esneklik sağlar .

Dosya Tipi Seçimi: Mümkünse yeni projelerde .pptx (XMLSlideShow) formatını kullanmanızı öneririm. Hem metin vurgulama gibi özellikler için desteği daha iyidir hem de daha modern bir yapı sunar.

Uygulamanızın genel mimarisini ve kullanmayı planladığınız PowerPoint dosya formatını (.ppt mi yoksa .pptx mi?) netleştirdikten sonra, kodlama aşamasında daha detaylı yardım isteyebilirsiniz. Kolay gelsin!

çizimler nasıl tutulacak ben backend tam anlamak istiyorum. pptx olacak. mimari şu şekilde olacak reactte uretilenler backend de apachi Poi ile işlenecek veya ne onerirsen. bunun yanında sayfa sayfa ekleyebilecegin bir yapı. 
Harika! Şimdi mimariyi daha net anladım. .pptx formatı ve Apache POI ile backend odaklı bir çözüm geliştireceğiz. İşte detaylı mimari ve çizimlerin nasıl tutulacağı:

📐 Mimari Tasarım
1. Veri Modeli (Backend)
Öncelikle React'ten gelecek çizimleri ve slaytları temsil edecek Java sınıfları oluşturalım:

java
// SlideElement.java - Slayttaki her bir çizim öğesini temsil eder
public class SlideElement {
    private String id;                    // Benzersiz ID
    private String type;                   // "rectangle", "circle", "line", "arrow", "highlight"
    private double x, y;                   // Konum
    private double width, height;           // Boyut
    private String color;                   // Renk (hex veya isim)
    private int strokeWidth;                 // Çizgi kalınlığı
    private String text;                     // Varsa metin içeriği
    private Map<String, Object> properties;  // Ek özellikler (doluluk, gölge vb.)
    
    // Getter ve Setter'lar
}

// Slide.java - Tek bir slaytı temsil eder
public class Slide {
    private int slideNumber;
    private List<SlideElement> elements;
    private SlideLayout layout;              // "title", "content", "blank" vb.
    private Map<String, Object> slideProperties;
    
    // Getter ve Setter'lar
}

// PresentationData.java - Tüm sunumu temsil eder
public class PresentationData {
    private String presentationId;
    private String presentationName;
    private List<Slide> slides;
    private PresentationMetadata metadata;
    
    // Getter ve Setter'lar
}

// PresentationMetadata.java
public class PresentationMetadata {
    private String author;
    private Date createdDate;
    private Date modifiedDate;
    private int slideCount;
    private Dimension pageSize;               // Slayt boyutu
}
2. Backend Servis Katmanı
java
@Service
public class PresentationService {
    
    private final SlideElementConverter elementConverter;
    private final SlideLayoutManager layoutManager;
    
    public PresentationService(SlideElementConverter elementConverter, 
                               SlideLayoutManager layoutManager) {
        this.elementConverter = elementConverter;
        this.layoutManager = layoutManager;
    }
    
    /**
     * React'ten gelen JSON verilerini PowerPoint'e dönüştürür
     */
    public byte[] createPresentation(PresentationData presentationData) throws Exception {
        try (XMLSlideShow ppt = new XMLSlideShow()) {
            
            // Slayt boyutunu ayarla (standart: 720x540 piksel)
            ppt.setPageSize(new Dimension(720, 540));
            
            // Her slayt için
            for (Slide slide : presentationData.getSlides()) {
                XSLFSlide xslfSlide = layoutManager.createSlide(ppt, slide.getLayout());
                
                // Slayttaki tüm elementleri işle
                for (SlideElement element : slide.getElements()) {
                    elementConverter.convertAndAdd(xslfSlide, element);
                }
            }
            
            // PowerPoint dosyasını byte array olarak döndür
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ppt.write(baos);
            return baos.toByteArray();
        }
    }
    
    /**
     * Varolan sunuma yeni slayt ekler
     */
    public byte[] addSlideToPresentation(byte[] existingPpt, Slide newSlide) throws Exception {
        try (XMLSlideShow ppt = new XMLSlideShow(new ByteArrayInputStream(existingPpt))) {
            XSLFSlide xslfSlide = ppt.createSlide();
            
            // Yeni slayttaki elementleri ekle
            for (SlideElement element : newSlide.getElements()) {
                elementConverter.convertAndAdd(xslfSlide, element);
            }
            
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ppt.write(baos);
            return baos.toByteArray();
        }
    }
}
3. Element Dönüştürücü (Çizimleri PowerPoint'e Çeviren Katman)
java
@Component
public class SlideElementConverter {
    
    private static final Map<String, BiConsumer<XSLFSlide, SlideElement>> CONVERTERS = new HashMap<>();
    
    static {
        CONVERTERS.put("rectangle", SlideElementConverter::addRectangle);
        CONVERTERS.put("circle", SlideElementConverter::addCircle);
        CONVERTERS.put("line", SlideElementConverter::addLine);
        CONVERTERS.put("arrow", SlideElementConverter::addArrow);
        CONVERTERS.put("text", SlideElementConverter::addText);
        CONVERTERS.put("highlight", SlideElementConverter::addHighlight);
        CONVERTERS.put("freeform", SlideElementConverter::addFreeform);
    }
    
    public void convertAndAdd(XSLFSlide slide, SlideElement element) {
        BiConsumer<XSLFSlide, SlideElement> converter = CONVERTERS.get(element.getType());
        if (converter != null) {
            converter.accept(slide, element);
        } else {
            throw new IllegalArgumentException("Bilinmeyen element tipi: " + element.getType());
        }
    }
    
    // Dikdörtgen ekle
    private static void addRectangle(XSLFSlide slide, SlideElement element) {
        XSLFAutoShape rect = slide.createAutoShape();
        rect.setShapeType(ShapeType.RECT);
        rect.setAnchor(new Rectangle(
            (int) element.getX(), 
            (int) element.getY(),
            (int) element.getWidth(), 
            (int) element.getHeight()
        ));
        
        // Renk ayarla
        if (element.getColor() != null) {
            rect.setFillColor(Color.decode(element.getColor()));
        }
        
        // Çizgi kalınlığı
        if (element.getStrokeWidth() > 0) {
            rect.setStrokeWeight(element.getStrokeWidth() / 10.0);
        }
        
        // Ek özellikler (gölge, 3B vb.)
        applyProperties(rect, element.getProperties());
    }
    
    // Çizgi ekle
    private static void addLine(XSLFSlide slide, SlideElement element) {
        XSLFAutoShape line = slide.createAutoShape();
        line.setShapeType(ShapeType.LINE);
        line.setAnchor(new Rectangle(
            (int) element.getX(), 
            (int) element.getY(),
            (int) element.getWidth(), 
            (int) element.getHeight()
        ));
        
        if (element.getColor() != null) {
            line.setLineColor(Color.decode(element.getColor()));
        }
        
        if (element.getStrokeWidth() > 0) {
            line.setStrokeWeight(element.getStrokeWidth() / 10.0);
        }
    }
    
    // Ok ekle (çizgi + ok başı)
    private static void addArrow(XSLFSlide slide, SlideElement element) {
        XSLFAutoShape arrow = slide.createAutoShape();
        arrow.setShapeType(ShapeType.ARROW);
        arrow.setAnchor(new Rectangle(
            (int) element.getX(), 
            (int) element.getY(),
            (int) element.getWidth(), 
            (int) element.getHeight()
        ));
        
        if (element.getColor() != null) {
            arrow.setLineColor(Color.decode(element.getColor()));
        }
    }
    
    // Metin kutusu
    private static void addText(XSLFSlide slide, SlideElement element) {
        XSLFTextBox textBox = slide.createTextBox();
        textBox.setAnchor(new Rectangle(
            (int) element.getX(), 
            (int) element.getY(),
            (int) element.getWidth(), 
            (int) element.getHeight()
        ));
        
        XSLFTextParagraph paragraph = textBox.addNewTextParagraph();
        XSLFTextRun run = paragraph.addNewTextRun();
        run.setText(element.getText() != null ? element.getText() : "");
        
        if (element.getColor() != null) {
            run.setFontColor(Color.decode(element.getColor()));
        }
    }
    
    // Vurgulama (highlight)
    private static void addHighlight(XSLFSlide slide, SlideElement element) {
        XSLFTextBox textBox = slide.createTextBox();
        textBox.setAnchor(new Rectangle(
            (int) element.getX(), 
            (int) element.getY(),
            (int) element.getWidth(), 
            (int) element.getHeight()
        ));
        
        XSLFTextParagraph paragraph = textBox.addNewTextParagraph();
        XSLFTextRun run = paragraph.addNewTextRun();
        run.setText(element.getText() != null ? element.getText() : "");
        
        // Metni vurgula
        if (element.getColor() != null) {
            run.setHighlightColor(Color.decode(element.getColor()));
        }
    }
    
    // Serbest el çizimi (nokta listesi ile)
    private static void addFreeform(XSLFSlide slide, SlideElement element) {
        // Nokta listesini properties'den al
        List<Map<String, Double>> points = (List<Map<String, Double>>) 
            element.getProperties().get("points");
        
        if (points != null && !points.isEmpty()) {
            // Java2D kullanarak serbest el çizimi
            // (Bu daha karmaşık bir yapı gerektirebilir)
        }
    }
    
    private static void applyProperties(XSLFAutoShape shape, Map<String, Object> properties) {
        if (properties == null) return;
        
        // Gölge efekti
        if (properties.containsKey("shadow")) {
            shape.setShadow(true);
            Map<String, Object> shadowProps = (Map<String, Object>) properties.get("shadow");
            // Gölge özelliklerini ayarla...
        }
        
        // Saydamlık
        if (properties.containsKey("opacity")) {
            double opacity = (double) properties.get("opacity");
            shape.setFillColor(new Color(0, 0, 0, (int)(255 * opacity)));
        }
        
        // Döndürme
        if (properties.containsKey("rotation")) {
            double rotation = (double) properties.get("rotation");
            shape.setRotation(rotation);
        }
    }
}
4. Controller Katmanı (REST API)
java
@RestController
@RequestMapping("/api/presentation")
@CrossOrigin(origins = "http://localhost:3000")
public class PresentationController {
    
    private final PresentationService presentationService;
    
    @PostMapping("/create")
    public ResponseEntity<byte[]> createPresentation(@RequestBody PresentationData presentationData) {
        try {
            byte[] pptBytes = presentationService.createPresentation(presentationData);
            
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
            headers.setContentDisposition(ContentDisposition.builder("attachment")
                .filename(presentationData.getPresentationName() + ".pptx")
                .build());
            
            return new ResponseEntity<>(pptBytes, headers, HttpStatus.OK);
        } catch (Exception e) {
            return ResponseEntity.internalServerError().build();
        }
    }
    
    @PostMapping("/add-slide/{presentationId}")
    public ResponseEntity<byte[]> addSlide(@PathVariable String presentationId, 
                                           @RequestBody Slide newSlide) {
        try {
            // Varolan sunuyu DB'den veya storage'dan al
            byte[] existingPpt = getPresentationFromStorage(presentationId);
            
            byte[] updatedPpt = presentationService.addSlideToPresentation(existingPpt, newSlide);
            
            // Güncellenmiş sunuyu kaydet
            savePresentationToStorage(presentationId, updatedPpt);
            
            return ResponseEntity.ok(updatedPpt);
        } catch (Exception e) {
            return ResponseEntity.internalServerError().build();
        }
    }
    
    private byte[] getPresentationFromStorage(String presentationId) {
        // DB'den veya dosya sisteminden sunuyu getir
        return null; // Implementasyon
    }
    
    private void savePresentationToStorage(String presentationId, byte[] pptData) {
        // Sunuyu DB'ye veya dosya sistemine kaydet
    }
}
5. React Tarafı (Örnek API Çağrısı)
javascript
// PresentationService.js
class PresentationService {
    async createPresentation(presentationData) {
        const response = await fetch('http://localhost:8080/api/presentation/create', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(presentationData)
        });
        
        if (response.ok) {
            // Dosyayı indir
            const blob = await response.blob();
            const url = window.URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `${presentationData.presentationName}.pptx`;
            a.click();
        }
    }
    
    async addSlide(presentationId, slideData) {
        const response = await fetch(`http://localhost:8080/api/presentation/add-slide/${presentationId}`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(slideData)
        });
        
        if (response.ok) {
            const updatedPpt = await response.blob();
            // Güncellenmiş sunuyu işle...
        }
    }
}
6. Veritabanı Şeması (İsteğe bağlı)
Eğer sunuları veritabanında saklamak isterseniz:

sql
-- Sunumlar tablosu
CREATE TABLE presentations (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(255),
    author VARCHAR(100),
    created_date TIMESTAMP,
    modified_date TIMESTAMP,
    slide_count INT,
    ppt_data LONGBLOB,  -- PowerPoint binary data
    metadata JSON
);

-- Slaytlar tablosu (her slayt için JSON)
CREATE TABLE slides (
    id VARCHAR(36) PRIMARY KEY,
    presentation_id VARCHAR(36),
    slide_number INT,
    layout VARCHAR(50),
    elements JSON,  -- Slayttaki tüm elementlerin JSON hali
    FOREIGN KEY (presentation_id) REFERENCES presentations(id)
);
7. Öneriler ve İpuçları
Asenkron İşlem: Büyük sunular için işlemleri asenkron yapın

Önbellekleme: Sık kullanılan şekilleri ve stilleri önbellekte tutun

Versiyon Kontrolü: Sunu versiyonlarını takip edin

Validation: Gelen verileri mutlaka validate edin

Hata Yönetimi: Her katmanda try-catch blokları kullanın

Bu mimari ile:

React'te her slayt için çizimler yapabilir

Her slaytı ayrı ayrı backend'e gönderebilir

Varolan sunulara yeni slaytlar ekleyebilir

Tüm çizim tiplerini (dikdörtgen, daire, çizgi, ok, highlight vb.) destekleyebilirsiniz

Başka merak ettiğiniz bir nokta var mı?

react için yapı nasıl olacak
Mükemmel! React tarafında kullanıcıların slaytlar üzerinde çizim yapabileceği, şekiller ekleyebileceği ve vurgulama yapabileceği bir arayüz kuracağız. İşte detaylı React mimarisi:

🎨 React Frontend Mimarisi
1. Proje Yapısı
text
src/
├── components/
│   ├── Presentation/
│   │   ├── PresentationEditor.jsx
│   │   ├── SlideCanvas.jsx
│   │   ├── SlideThumbnail.jsx
│   │   ├── SlideList.jsx
│   │   └── SlideControls.jsx
│   ├── DrawingTools/
│   │   ├── Toolbar.jsx
│   │   ├── ShapeTools.jsx
│   │   ├── ColorPicker.jsx
│   │   └── TextTools.jsx
│   ├── Properties/
│   │   ├── PropertyPanel.jsx
│   │   └── ElementProperties.jsx
│   └── Common/
│       ├── Modal.jsx
│       └── Button.jsx
├── hooks/
│   ├── useDrawing.js
│   ├── useSlides.js
│   └── usePresentation.js
├── services/
│   ├── apiService.js
│   └── presentationService.js
├── store/
│   ├── presentationSlice.js
│   └── store.js
├── utils/
│   ├── shapeUtils.js
│   ├── colorUtils.js
│   └── exportUtils.js
└── styles/
    └── editor.css
2. Temel Bileşenler
PresentationEditor.jsx - Ana Editor Bileşeni
jsx
import React, { useState, useCallback } from 'react';
import SlideCanvas from './SlideCanvas';
import Toolbar from '../DrawingTools/Toolbar';
import SlideList from './SlideList';
import PropertyPanel from '../Properties/PropertyPanel';
import { useSlides } from '../../hooks/useSlides';
import { usePresentation } from '../../hooks/usePresentation';
import './PresentationEditor.css';

const PresentationEditor = () => {
  const [currentSlideIndex, setCurrentSlideIndex] = useState(0);
  const [selectedTool, setSelectedTool] = useState('select');
  const [selectedElement, setSelectedElement] = useState(null);
  
  const { 
    slides, 
    addSlide, 
    removeSlide, 
    updateSlide,
    duplicateSlide 
  } = useSlides();
  
  const { 
    exportPresentation, 
    savePresentation,
    importPresentation 
  } = usePresentation();

  const handleElementAdd = useCallback((element) => {
    const updatedSlides = [...slides];
    if (!updatedSlides[currentSlideIndex].elements) {
      updatedSlides[currentSlideIndex].elements = [];
    }
    updatedSlides[currentSlideIndex].elements.push({
      ...element,
      id: `elem_${Date.now()}_${Math.random()}`
    });
    updateSlide(currentSlideIndex, updatedSlides[currentSlideIndex]);
  }, [slides, currentSlideIndex, updateSlide]);

  const handleElementUpdate = useCallback((elementId, updates) => {
    const updatedSlides = [...slides];
    const elements = updatedSlides[currentSlideIndex].elements;
    const index = elements.findIndex(el => el.id === elementId);
    if (index !== -1) {
      elements[index] = { ...elements[index], ...updates };
      updateSlide(currentSlideIndex, updatedSlides[currentSlideIndex]);
    }
  }, [slides, currentSlideIndex, updateSlide]);

  const handleElementDelete = useCallback((elementId) => {
    const updatedSlides = [...slides];
    updatedSlides[currentSlideIndex].elements = 
      updatedSlides[currentSlideIndex].elements.filter(el => el.id !== elementId);
    updateSlide(currentSlideIndex, updatedSlides[currentSlideIndex]);
    setSelectedElement(null);
  }, [slides, currentSlideIndex, updateSlide]);

  return (
    <div className="presentation-editor">
      <Toolbar 
        selectedTool={selectedTool}
        onToolSelect={setSelectedTool}
        onAddSlide={addSlide}
        onSave={savePresentation}
        onExport={exportPresentation}
      />
      
      <div className="editor-main">
        <SlideList 
          slides={slides}
          currentSlide={currentSlideIndex}
          onSlideSelect={setCurrentSlideIndex}
          onSlideAdd={addSlide}
          onSlideRemove={removeSlide}
          onSlideDuplicate={duplicateSlide}
        />
        
        <div className="canvas-container">
          <SlideCanvas
            slide={slides[currentSlideIndex]}
            selectedTool={selectedTool}
            selectedElement={selectedElement}
            onElementSelect={setSelectedElement}
            onElementAdd={handleElementAdd}
            onElementUpdate={handleElementUpdate}
            onElementDelete={handleElementDelete}
          />
        </div>
        
        <PropertyPanel
          selectedElement={selectedElement}
          onElementUpdate={handleElementUpdate}
          slideProperties={slides[currentSlideIndex]}
          onSlideUpdate={(updates) => updateSlide(currentSlideIndex, updates)}
        />
      </div>
    </div>
  );
};

export default PresentationEditor;
SlideCanvas.jsx - Fabric.js ile Çizim Alanı
jsx
import React, { useRef, useEffect, useState } from 'react';
import { fabric } from 'fabric';
import './SlideCanvas.css';

const SlideCanvas = ({ 
  slide, 
  selectedTool, 
  selectedElement,
  onElementSelect,
  onElementAdd,
  onElementUpdate,
  onElementDelete 
}) => {
  const canvasRef = useRef(null);
  const fabricCanvasRef = useRef(null);
  const [isDrawing, setIsDrawing] = useState(false);

  // Canvas'ı başlat
  useEffect(() => {
    const canvas = new fabric.Canvas(canvasRef.current, {
      width: 720,
      height: 540,
      backgroundColor: 'white',
      preserveObjectStacking: true
    });

    fabricCanvasRef.current = canvas;

    // Olay dinleyicileri
    canvas.on('object:added', handleObjectAdded);
    canvas.on('object:modified', handleObjectModified);
    canvas.on('selection:created', handleSelection);
    canvas.on('selection:updated', handleSelection);
    canvas.on('selection:cleared', () => onElementSelect(null));

    // Klavye kısayolları
    canvas.on('object:removed', () => {});

    return () => {
      canvas.dispose();
    };
  }, []);

  // Slayt değiştiğinde canvas'ı güncelle
  useEffect(() => {
    if (fabricCanvasRef.current && slide) {
      loadSlideToCanvas(slide);
    }
  }, [slide]);

  // Seçili araca göre çizim modunu ayarla
  useEffect(() => {
    const canvas = fabricCanvasRef.current;
    if (!canvas) return;

    // Tüm çizim modlarını kapat
    canvas.isDrawingMode = false;
    canvas.selection = true;
    canvas.defaultCursor = 'default';

    switch (selectedTool) {
      case 'select':
        canvas.selection = true;
        canvas.defaultCursor = 'default';
        break;
        
      case 'rectangle':
        canvas.selection = false;
        canvas.defaultCursor = 'crosshair';
        enableShapeDrawing('rectangle');
        break;
        
      case 'circle':
        canvas.selection = false;
        canvas.defaultCursor = 'crosshair';
        enableShapeDrawing('circle');
        break;
        
      case 'line':
        canvas.selection = false;
        canvas.defaultCursor = 'crosshair';
        enableLineDrawing();
        break;
        
      case 'arrow':
        canvas.selection = false;
        canvas.defaultCursor = 'crosshair';
        enableArrowDrawing();
        break;
        
      case 'text':
        canvas.selection = false;
        canvas.defaultCursor = 'text';
        enableTextDrawing();
        break;
        
      case 'highlight':
        canvas.selection = false;
        canvas.defaultCursor = 'crosshair';
        enableHighlightMode();
        break;
        
      case 'freehand':
        canvas.isDrawingMode = true;
        canvas.freeDrawingBrush = new fabric.PencilBrush(canvas);
        canvas.freeDrawingBrush.color = '#000000';
        canvas.freeDrawingBrush.width = 2;
        break;
        
      default:
        break;
    }
  }, [selectedTool]);

  // Şekil çizimini etkinleştir
  const enableShapeDrawing = (shapeType) => {
    const canvas = fabricCanvasRef.current;
    
    canvas.on('mouse:down', (options) => {
      if (!options.target) {
        const pointer = canvas.getPointer(options.e);
        startDrawingShape(shapeType, pointer);
      }
    });
  };

  const startDrawingShape = (shapeType, pointer) => {
    const canvas = fabricCanvasRef.current;
    let shape;

    switch (shapeType) {
      case 'rectangle':
        shape = new fabric.Rect({
          left: pointer.x,
          top: pointer.y,
          width: 1,
          height: 1,
          fill: 'transparent',
          stroke: '#000000',
          strokeWidth: 2
        });
        break;
        
      case 'circle':
        shape = new fabric.Circle({
          left: pointer.x,
          top: pointer.y,
          radius: 1,
          fill: 'transparent',
          stroke: '#000000',
          strokeWidth: 2
        });
        break;
    }

    if (shape) {
      canvas.add(shape);
      canvas.setActiveObject(shape);
      
      // Şekli yeniden boyutlandırma
      const handleMouseMove = (e) => {
        const pointer = canvas.getPointer(e.e);
        const width = Math.abs(pointer.x - shape.left);
        const height = Math.abs(pointer.y - shape.top);
        
        if (shapeType === 'rectangle') {
          shape.set({ width, height });
        } else if (shapeType === 'circle') {
          shape.set({ radius: Math.max(width, height) / 2 });
        }
        
        canvas.renderAll();
      };

      const handleMouseUp = () => {
        canvas.off('mouse:move', handleMouseMove);
        canvas.off('mouse:up', handleMouseUp);
        
        // Elementi kaydet
        const element = convertFabricObjectToElement(shape);
        onElementAdd(element);
      };

      canvas.on('mouse:move', handleMouseMove);
      canvas.on('mouse:up', handleMouseUp);
    }
  };

  // Metin ekleme
  const enableTextDrawing = () => {
    const canvas = fabricCanvasRef.current;
    
    canvas.on('mouse:down', (options) => {
      if (!options.target) {
        const pointer = canvas.getPointer(options.e);
        
        const text = new fabric.IText('Metin girin', {
          left: pointer.x,
          top: pointer.y,
          fontSize: 20,
          fontFamily: 'Arial',
          fill: '#000000'
        });
        
        canvas.add(text);
        canvas.setActiveObject(text);
        text.enterEditing();
        
        const element = convertFabricObjectToElement(text);
        onElementAdd(element);
      }
    });
  };

  // Vurgulama modu
  const enableHighlightMode = () => {
    const canvas = fabricCanvasRef.current;
    
    canvas.on('mouse:down', (options) => {
      if (options.target && options.target.type === 'i-text') {
        const text = options.target;
        const selectionStart = text.selectionStart || 0;
        const selectionEnd = text.selectionEnd || text.text.length;
        
        if (selectionStart !== selectionEnd) {
          // Seçili metni vurgula
          const highlight = new fabric.Rect({
            left: text.left,
            top: text.top,
            width: text.width,
            height: text.height,
            fill: 'rgba(255, 255, 0, 0.3)',
            selectable: true
          });
          
          canvas.add(highlight);
          canvas.sendToBack(highlight);
          
          const element = {
            id: `highlight_${Date.now()}`,
            type: 'highlight',
            x: highlight.left,
            y: highlight.top,
            width: highlight.width,
            height: highlight.height,
            color: '#FFFF00',
            text: text.text.substring(selectionStart, selectionEnd)
          };
          
          onElementAdd(element);
        }
      }
    });
  };

  // Canvas'a slayt yükle
  const loadSlideToCanvas = (slide) => {
    const canvas = fabricCanvasRef.current;
    canvas.clear();
    canvas.backgroundColor = 'white';
    
    if (slide.elements && slide.elements.length > 0) {
      slide.elements.forEach(element => {
        const fabricObject = convertElementToFabricObject(element);
        if (fabricObject) {
          canvas.add(fabricObject);
        }
      });
    }
    
    canvas.renderAll();
  };

  // Element dönüştürücüler
  const convertElementToFabricObject = (element) => {
    switch (element.type) {
      case 'rectangle':
        return new fabric.Rect({
          left: element.x,
          top: element.y,
          width: element.width,
          height: element.height,
          fill: element.color || 'transparent',
          stroke: '#000000',
          strokeWidth: element.strokeWidth || 2,
          id: element.id
        });
        
      case 'circle':
        return new fabric.Circle({
          left: element.x,
          top: element.y,
          radius: element.width / 2,
          fill: element.color || 'transparent',
          stroke: '#000000',
          strokeWidth: element.strokeWidth || 2,
          id: element.id
        });
        
      case 'text':
        return new fabric.IText(element.text || '', {
          left: element.x,
          top: element.y,
          fontSize: element.properties?.fontSize || 20,
          fontFamily: element.properties?.fontFamily || 'Arial',
          fill: element.color || '#000000',
          id: element.id
        });
        
      case 'highlight':
        return new fabric.Rect({
          left: element.x,
          top: element.y,
          width: element.width,
          height: element.height,
          fill: element.color || 'rgba(255, 255, 0, 0.3)',
          selectable: true,
          id: element.id
        });
        
      default:
        return null;
    }
  };

  const convertFabricObjectToElement = (obj) => {
    const baseElement = {
      id: obj.id || `elem_${Date.now()}`,
      x: obj.left,
      y: obj.top,
      width: obj.width,
      height: obj.height,
      color: obj.fill,
      strokeWidth: obj.strokeWidth,
      properties: {}
    };

    if (obj.type === 'rect') {
      return { ...baseElement, type: 'rectangle' };
    } else if (obj.type === 'circle') {
      return { ...baseElement, type: 'circle', width: obj.radius * 2 };
    } else if (obj.type === 'i-text') {
      return { 
        ...baseElement, 
        type: 'text', 
        text: obj.text,
        properties: {
          fontSize: obj.fontSize,
          fontFamily: obj.fontFamily
        }
      };
    }

    return baseElement;
  };

  // Olay işleyiciler
  const handleObjectAdded = (e) => {
    // Yeni nesne eklendi
  };

  const handleObjectModified = (e) => {
    const obj = e.target;
    if (obj.id) {
      const element = convertFabricObjectToElement(obj);
      onElementUpdate(obj.id, element);
    }
  };

  const handleSelection = (e) => {
    if (e.selected && e.selected.length > 0) {
      const obj = e.selected[0];
      onElementSelect({
        id: obj.id,
        type: obj.type,
        ...obj.toJSON()
      });
    }
  };

  return (
    <div className="slide-canvas">
      <canvas ref={canvasRef} />
      <div className="canvas-overlay">
        {/* Slayt numarası göstergesi */}
        <div className="slide-number">
          Slayt {slide?.slideNumber || 1}
        </div>
      </div>
    </div>
  );
};

export default SlideCanvas;
Toolbar.jsx - Araç Çubuğu
jsx
import React from 'react';
import './Toolbar.css';

const Toolbar = ({ selectedTool, onToolSelect, onAddSlide, onSave, onExport }) => {
  const tools = [
    { id: 'select', icon: '🖱️', label: 'Seç' },
    { id: 'rectangle', icon: '⬜', label: 'Dikdörtgen' },
    { id: 'circle', icon: '⭕', label: 'Daire' },
    { id: 'line', icon: '📏', label: 'Çizgi' },
    { id: 'arrow', icon: '➡️', label: 'Ok' },
    { id: 'text', icon: '📝', label: 'Metin' },
    { id: 'highlight', icon: '🟨', label: 'Vurgula' },
    { id: 'freehand', icon: '✏️', label: 'Serbest Çizim' }
  ];

  return (
    <div className="toolbar">
      <div className="toolbar-group">
        {tools.map(tool => (
          <button
            key={tool.id}
            className={`tool-button ${selectedTool === tool.id ? 'active' : ''}`}
            onClick={() => onToolSelect(tool.id)}
            title={tool.label}
          >
            <span className="tool-icon">{tool.icon}</span>
          </button>
        ))}
      </div>

      <div className="toolbar-group">
        <button className="tool-button" onClick={onAddSlide} title="Yeni Slayt">
          <span className="tool-icon">➕</span>
        </button>
        <button className="tool-button" onClick={onSave} title="Kaydet">
          <span className="tool-icon">💾</span>
        </button>
        <button className="tool-button" onClick={onExport} title="PPTX Olarak Dışa Aktar">
          <span className="tool-icon">📊</span>
        </button>
      </div>
    </div>
  );
};

export default Toolbar;
SlideList.jsx - Slayt Listesi
jsx
import React from 'react';
import SlideThumbnail from './SlideThumbnail';
import './SlideList.css';

const SlideList = ({ 
  slides, 
  currentSlide, 
  onSlideSelect, 
  onSlideAdd, 
  onSlideRemove,
  onSlideDuplicate 
}) => {
  return (
    <div className="slide-list">
      <div className="slide-list-header">
        <h3>Slaytlar</h3>
        <button onClick={onSlideAdd} className="add-slide-btn">+</button>
      </div>
      
      <div className="slide-thumbnails">
        {slides.map((slide, index) => (
          <SlideThumbnail
            key={slide.id || index}
            slide={slide}
            index={index}
            isSelected={index === currentSlide}
            onClick={() => onSlideSelect(index)}
            onDuplicate={() => onSlideDuplicate(index)}
            onDelete={() => onSlideRemove(index)}
          />
        ))}
      </div>
    </div>
  );
};

export default SlideList;
3. Custom Hooks
useSlides.js - Slayt Yönetimi
javascript
import { useState, useCallback } from 'react';

export const useSlides = () => {
  const [slides, setSlides] = useState([
    {
      id: `slide_${Date.now()}`,
      slideNumber: 1,
      layout: 'blank',
      elements: [],
      backgroundColor: '#ffffff'
    }
  ]);

  const addSlide = useCallback(() => {
    setSlides(prev => [
      ...prev,
      {
        id: `slide_${Date.now()}`,
        slideNumber: prev.length + 1,
        layout: 'blank',
        elements: [],
        backgroundColor: '#ffffff'
      }
    ]);
  }, []);

  const removeSlide = useCallback((index) => {
    setSlides(prev => prev.filter((_, i) => i !== index));
  }, []);

  const duplicateSlide = useCallback((index) => {
    setSlides(prev => {
      const slideToDuplicate = prev[index];
      const duplicatedSlide = {
        ...slideToDuplicate,
        id: `slide_${Date.now()}`,
        slideNumber: prev.length + 1,
        elements: slideToDuplicate.elements.map(el => ({
          ...el,
          id: `elem_${Date.now()}_${Math.random()}`
        }))
      };
      
      return [
        ...prev.slice(0, index + 1),
        duplicatedSlide,
        ...prev.slice(index + 1)
      ];
    });
  }, []);

  const updateSlide = useCallback((index, updatedSlide) => {
    setSlides(prev => {
      const newSlides = [...prev];
      newSlides[index] = updatedSlide;
      return newSlides;
    });
  }, []);

  return {
    slides,
    addSlide,
    removeSlide,
    duplicateSlide,
    updateSlide
  };
};
usePresentation.js - Sunum Yönetimi
javascript
import { useCallback } from 'react';
import presentationService from '../services/presentationService';

export const usePresentation = () => {
  const exportPresentation = useCallback(async (slides, name) => {
    try {
      const presentationData = {
        presentationId: `pres_${Date.now()}`,
        presentationName: name || 'Sunum',
        slides: slides.map((slide, index) => ({
          slideNumber: index + 1,
          layout: slide.layout || 'blank',
          elements: slide.elements || []
        })),
        metadata: {
          author: 'Kullanıcı',
          createdDate: new Date(),
          slideCount: slides.length
        }
      };

      await presentationService.createPresentation(presentationData);
    } catch (error) {
      console.error('Sunum dışa aktarılırken hata:', error);
    }
  }, []);

  const savePresentation = useCallback(async (slides, name) => {
    try {
      localStorage.setItem('presentation', JSON.stringify({
        name,
        slides,
        lastModified: new Date()
      }));
    } catch (error) {
      console.error('Sunum kaydedilirken hata:', error);
    }
  }, []);

  const importPresentation = useCallback(async () => {
    try {
      const saved = localStorage.getItem('presentation');
      return saved ? JSON.parse(saved) : null;
    } catch (error) {
      console.error('Sunum yüklenirken hata:', error);
      return null;
    }
  }, []);

  return {
    exportPresentation,
    savePresentation,
    importPresentation
  };
};
4. API Service
presentationService.js
javascript
class PresentationService {
  constructor() {
    this.baseUrl = process.env.REACT_APP_API_URL || 'http://localhost:8080/api';
  }

  async createPresentation(presentationData) {
    const response = await fetch(`${this.baseUrl}/presentation/create`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(presentationData)
    });

    if (!response.ok) {
      throw new Error('Sunum oluşturulamadı');
    }

    // Dosyayı indir
    const blob = await response.blob();
    const url = window.URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.download = `${presentationData.presentationName}.pptx`;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    window.URL.revokeObjectURL(url);
  }

  async addSlide(presentationId, slideData) {
    const response = await fetch(`${this.baseUrl}/presentation/add-slide/${presentationId}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(slideData)
    });

    if (!response.ok) {
      throw new Error('Slayt eklenemedi');
    }

    return response.blob();
  }
}

export default new PresentationService();
5. Stil Dosyası (CSS)
css
/* PresentationEditor.css */
.presentation-editor {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background-color: #f0f0f0;
}

.editor-main {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.canvas-container {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  background-color: #e0e0e0;
}

/* SlideCanvas.css */
.slide-canvas {
  position: relative;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  background-color: white;
}

.slide-canvas canvas {
  display: block;
  border: 1px solid #ccc;
}

.canvas-overlay {
  position: absolute;
  bottom: 10px;
  right: 10px;
  background-color: rgba(0, 0, 0, 0.6);
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
}

/* Toolbar.css */
.toolbar {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  background-color: white;
  border-bottom: 1px solid #ddd;
}

.toolbar-group {
  display: flex;
  gap: 5px;
}

.tool-button {
  width: 40px;
  height: 40px;
  border: 1px solid #ddd;
  background-color: white;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
}

.tool-button:hover {
  background-color: #f5f5f5;
}

.tool-button.active {
  background-color: #e3f2fd;
  border-color: #2196f3;
}

.tool-icon {
  font-size: 20px;
}

/* SlideList.css */
.slide-list {
  width: 200px;
  background-color: white;
  border-right: 1px solid #ddd;
  overflow-y: auto;
}

.slide-list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  border-bottom: 1px solid #ddd;
}

.slide-thumbnails {
  padding: 10px;
}

.slide-thumbnail {
  position: relative;
  margin-bottom: 10px;
  cursor: pointer;
  border: 2px solid transparent;
  transition: border-color 0.2s;
}

.slide-thumbnail.selected {
  border-color: #2196f3;
}

.thumbnail-content {
  height: 120px;
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  color: #666;
}

.thumbnail-actions {
  position: absolute;
  top: 5px;
  right: 5px;
  display: flex;
  gap: 5px;
  opacity: 0;
  transition: opacity 0.2s;
}

.slide-thumbnail:hover .thumbnail-actions {
  opacity: 1;
}

.thumbnail-actions button {
  width: 24px;
  height: 24px;
  border: none;
  background-color: rgba(255, 255, 255, 0.9);
  border-radius: 4px;
  cursor: pointer;
}
6. Örnek Kullanım
jsx
// App.js
import React from 'react';
import PresentationEditor from './components/Presentation/PresentationEditor';

function App() {
  return (
    <div className="App">
      <PresentationEditor />
    </div>
  );
}

export default App;
7. Gerekli Paketler
json
{
  "dependencies": {
    "react": "^18.2.0",
    "fabric": "^5.3.0",
    "axios": "^1.4.0",
    "react-color": "^2.19.3",
    "lodash": "^4.17.21",
    "@reduxjs/toolkit": "^1.9.5",
    "react-redux": "^8.1.1"
  }
}
Bu yapı ile:

Fabric.js kullanarak zengin çizim özellikleri

Slaytlar arası geçiş ve yönetim

Her türlü şekil, metin ve vurgulama desteği

Backend ile entegrasyon

Yerel kaydetme ve dışa aktarma

Özelliklerine sahip profesyonel bir PowerPoint benzeri editor elde edeceksiniz. İhtiyacınız olan ek özellikler varsa söyleyin, onları da ekleyelim!

nesnelerin sıralaması daha öne gelmesi daha arkaya gitmesi yine video resim vs interaktif herşeyin eklenebilmesi hatta bunları bir yere kaydedip sonra tekrar tekrar kullanabilmesi istiyirum
Harika! Şimdi profesyonel bir sunum editörü için gereken tüm özellikleri ekleyeceğiz. İşte gelişmiş katman yönetimi, medya desteği ve şablon sistemi içeren kapsamlı bir çözüm:

🚀 Gelişmiş React Sunum Editörü
1. Genişletilmiş Veri Modeli
javascript
// types/elementTypes.js
export const ElementTypes = {
  SHAPE: 'shape',
  TEXT: 'text',
  IMAGE: 'image',
  VIDEO: 'video',
  AUDIO: 'audio',
  CHART: 'chart',
  TABLE: 'table',
  ICON: 'icon',
  GROUP: 'group',
  HYPERLINK: 'hyperlink',
  EMBED: 'embed'
};

export const ShapeTypes = {
  RECTANGLE: 'rectangle',
  CIRCLE: 'circle',
  TRIANGLE: 'triangle',
  LINE: 'line',
  ARROW: 'arrow',
  CLOUD: 'cloud',
  STAR: 'star',
  HEART: 'heart',
  CALLOUT: 'callout',
  CUBE: 'cube',
  CYLINDER: 'cylinder'
};

// types/element.interface.js
export class SlideElement {
  constructor({
    id = `elem_${Date.now()}_${Math.random()}`,
    type = ElementTypes.SHAPE,
    shapeType = ShapeTypes.RECTANGLE,
    x = 0,
    y = 0,
    width = 100,
    height = 100,
    rotation = 0,
    scaleX = 1,
    scaleY = 1,
    opacity = 1,
    visible = true,
    locked = false,
    layer = 0,
    name = '',
    styles = {},
    content = null,
    mediaInfo = null,
    animation = null,
    hyperlink = null,
    groupId = null,
    metadata = {},
    effects = [],
    crop = null,
    flipX = false,
    flipY = false
  } = {}) {
    this.id = id;
    this.type = type;
    this.shapeType = shapeType;
    this.x = x;
    this.y = y;
    this.width = width;
    this.height = height;
    this.rotation = rotation;
    this.scaleX = scaleX;
    this.scaleY = scaleY;
    this.opacity = opacity;
    this.visible = visible;
    this.locked = locked;
    this.layer = layer;
    this.name = name;
    this.styles = styles;
    this.content = content;
    this.mediaInfo = mediaInfo;
    this.animation = animation;
    this.hyperlink = hyperlink;
    this.groupId = groupId;
    this.metadata = metadata;
    this.effects = effects;
    this.crop = crop;
    this.flipX = flipX;
    this.flipY = flipY;
  }
}
2. Gelişmiş SlideCanvas Bileşeni
jsx
// components/Presentation/AdvancedSlideCanvas.jsx
import React, { useRef, useEffect, useState, useCallback } from 'react';
import { fabric } from 'fabric';
import VideoElement from '../Media/VideoElement';
import AudioElement from '../Media/AudioElement';
import ChartElement from '../Charts/ChartElement';
import { ElementTypes } from '../../types/elementTypes';
import './AdvancedSlideCanvas.css';

const AdvancedSlideCanvas = ({
  slide,
  selectedTool,
  selectedElement,
  onElementSelect,
  onElementAdd,
  onElementUpdate,
  onElementDelete,
  onLayerChange,
  onGroupElements,
  onUngroupElements
}) => {
  const canvasRef = useRef(null);
  const fabricCanvasRef = useRef(null);
  const [clipboard, setClipboard] = useState(null);
  const [history, setHistory] = useState([]);
  const [historyIndex, setHistoryIndex] = useState(-1);

  // Canvas'ı başlat
  useEffect(() => {
    const canvas = new fabric.Canvas(canvasRef.current, {
      width: 1280,
      height: 720,
      backgroundColor: '#ffffff',
      preserveObjectStacking: true,
      allowTouchScrolling: true,
      selection: true,
      centeredRotation: true,
      centeredScaling: true
    });

    // Özel medya öğeleri için destek
    fabric.VideoElement = fabric.util.createClass(fabric.Object, {
      type: 'video',
      initialize: function(options) {
        this.callSuper('initialize', options);
        this.videoElement = document.createElement('video');
        this.videoElement.src = options.src;
        this.videoElement.controls = true;
        this.videoElement.style.width = '100%';
        this.videoElement.style.height = '100%';
      },
      _render: function(ctx) {
        this.callSuper('_render', ctx);
        if (this.videoElement) {
          ctx.drawImage(this.videoElement, -this.width/2, -this.height/2, this.width, this.height);
        }
      }
    });

    fabricCanvasRef.current = canvas;
    setupEventListeners(canvas);
    setupKeyboardShortcuts(canvas);

    return () => {
      canvas.dispose();
    };
  }, []);

  // Olay dinleyicileri
  const setupEventListeners = (canvas) => {
    canvas.on('object:added', (e) => addToHistory(e));
    canvas.on('object:modified', (e) => addToHistory(e));
    canvas.on('object:removed', (e) => addToHistory(e));
    canvas.on('selection:created', (e) => handleSelection(e));
    canvas.on('selection:updated', (e) => handleSelection(e));
    canvas.on('selection:cleared', () => onElementSelect(null));
  };

  // Klavye kısayolları
  const setupKeyboardShortcuts = (canvas) => {
    document.addEventListener('keydown', (e) => {
      // Ctrl+C (Kopyala)
      if (e.ctrlKey && e.key === 'c') {
        e.preventDefault();
        copySelected();
      }
      // Ctrl+V (Yapıştır)
      else if (e.ctrlKey && e.key === 'v') {
        e.preventDefault();
        pasteClipboard();
      }
      // Ctrl+X (Kes)
      else if (e.ctrlKey && e.key === 'x') {
        e.preventDefault();
        cutSelected();
      }
      // Ctrl+D (Çoğalt)
      else if (e.ctrlKey && e.key === 'd') {
        e.preventDefault();
        duplicateSelected();
      }
      // Ctrl+G (Grupla)
      else if (e.ctrlKey && e.key === 'g') {
        e.preventDefault();
        groupSelected();
      }
      // Ctrl+Shift+G (Grubu çöz)
      else if (e.ctrlKey && e.shiftKey && e.key === 'g') {
        e.preventDefault();
        ungroupSelected();
      }
      // Delete (Sil)
      else if (e.key === 'Delete') {
        e.preventDefault();
        deleteSelected();
      }
      // Ctrl+Z (Geri al)
      else if (e.ctrlKey && e.key === 'z') {
        e.preventDefault();
        undo();
      }
      // Ctrl+Y (İleri al)
      else if (e.ctrlKey && e.key === 'y') {
        e.preventDefault();
        redo();
      }
      // Ok tuşları ile hareket
      else if (e.key.startsWith('Arrow')) {
        const activeObject = canvas.getActiveObject();
        if (activeObject) {
          e.preventDefault();
          const step = e.shiftKey ? 10 : 1;
          const movement = {
            ArrowLeft: { left: -step },
            ArrowRight: { left: step },
            ArrowUp: { top: -step },
            ArrowDown: { top: step }
          }[e.key];
          
          activeObject.set(movement);
          activeObject.setCoords();
          canvas.renderAll();
          updateElementFromObject(activeObject);
        }
      }
    });
  };

  // Medya ekleme
  const addMedia = useCallback(async (file, type) => {
    const canvas = fabricCanvasRef.current;
    
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      
      reader.onload = async (e) => {
        try {
          let fabricObject;
          
          switch (type) {
            case 'image':
              fabric.Image.fromURL(e.target.result, (img) => {
                img.set({
                  left: 100,
                  top: 100,
                  scaleX: 0.5,
                  scaleY: 0.5,
                  hasControls: true,
                  hasBorders: true
                });
                canvas.add(img);
                
                const element = convertFabricObjectToElement(img, ElementTypes.IMAGE);
                onElementAdd(element);
                resolve(element);
              });
              break;
              
            case 'video':
              // Özel video objesi oluştur
              const videoElement = document.createElement('video');
              videoElement.src = e.target.result;
              videoElement.controls = true;
              
              const videoObj = new fabric.Rect({
                left: 100,
                top: 100,
                width: 320,
                height: 240,
                fill: '#2c3e50',
                stroke: '#3498db',
                strokeWidth: 2,
                rx: 10,
                ry: 10
              });
              
              // Video metadatası
              videoObj.videoInfo = {
                src: e.target.result,
                type: file.type,
                duration: 0
              };
              
              canvas.add(videoObj);
              
              const element = convertFabricObjectToElement(videoObj, ElementTypes.VIDEO);
              element.mediaInfo = {
                src: e.target.result,
                type: file.type
              };
              onElementAdd(element);
              resolve(element);
              break;
              
            case 'audio':
              const audioObj = new fabric.Rect({
                left: 100,
                top: 100,
                width: 300,
                height: 80,
                fill: '#27ae60',
                stroke: '#2ecc71',
                strokeWidth: 2,
                rx: 10,
                ry: 10
              });
              
              audioObj.audioInfo = {
                src: e.target.result,
                type: file.type
              };
              
              canvas.add(audioObj);
              
              const audioElement = convertFabricObjectToElement(audioObj, ElementTypes.AUDIO);
              audioElement.mediaInfo = {
                src: e.target.result,
                type: file.type
              };
              onElementAdd(audioElement);
              resolve(audioElement);
              break;
              
            default:
              reject(new Error('Desteklenmeyen medya tipi'));
          }
        } catch (error) {
          reject(error);
        }
      };
      
      reader.readAsDataURL(file);
    });
  }, [onElementAdd]);

  // Katman yönetimi
  const bringForward = useCallback(() => {
    const canvas = fabricCanvasRef.current;
    const activeObject = canvas.getActiveObject();
    
    if (activeObject) {
      canvas.bringForward(activeObject);
      canvas.renderAll();
      
      const element = convertFabricObjectToElement(activeObject);
      element.layer = canvas.getObjects().indexOf(activeObject);
      onElementUpdate(activeObject.id, element);
      onLayerChange('forward', element);
    }
  }, [onElementUpdate, onLayerChange]);

  const sendBackward = useCallback(() => {
    const canvas = fabricCanvasRef.current;
    const activeObject = canvas.getActiveObject();
    
    if (activeObject) {
      canvas.sendBackwards(activeObject);
      canvas.renderAll();
      
      const element = convertFabricObjectToElement(activeObject);
      element.layer = canvas.getObjects().indexOf(activeObject);
      onElementUpdate(activeObject.id, element);
      onLayerChange('backward', element);
    }
  }, [onElementUpdate, onLayerChange]);

  const bringToFront = useCallback(() => {
    const canvas = fabricCanvasRef.current;
    const activeObject = canvas.getActiveObject();
    
    if (activeObject) {
      canvas.bringToFront(activeObject);
      canvas.renderAll();
      
      const element = convertFabricObjectToElement(activeObject);
      element.layer = canvas.getObjects().indexOf(activeObject);
      onElementUpdate(activeObject.id, element);
      onLayerChange('front', element);
    }
  }, [onElementUpdate, onLayerChange]);

  const sendToBack = useCallback(() => {
    const canvas = fabricCanvasRef.current;
    const activeObject = canvas.getActiveObject();
    
    if (activeObject) {
      canvas.sendToBack(activeObject);
      canvas.renderAll();
      
      const element = convertFabricObjectToElement(activeObject);
      element.layer = canvas.getObjects().indexOf(activeObject);
      onElementUpdate(activeObject.id, element);
      onLayerChange('back', element);
    }
  }, [onElementUpdate, onLayerChange]);

  // Gruplama işlemleri
  const groupSelected = useCallback(() => {
    const canvas = fabricCanvasRef.current;
    const activeObjects = canvas.getActiveObjects();
    
    if (activeObjects.length > 1) {
      const group = new fabric.Group(activeObjects, {
        id: `group_${Date.now()}`,
        hasControls: true,
        hasBorders: true
      });
      
      canvas.discardActiveObject();
      activeObjects.forEach(obj => canvas.remove(obj));
      canvas.add(group);
      canvas.setActiveObject(group);
      canvas.renderAll();
      
      const element = convertFabricObjectToElement(group, ElementTypes.GROUP);
      element.groupId = group.id;
      onElementAdd(element);
      onGroupElements(activeObjects.map(obj => obj.id));
    }
  }, [onElementAdd, onGroupElements]);

  const ungroupSelected = useCallback(() => {
    const canvas = fabricCanvasRef.current;
    const activeObject = canvas.getActiveObject();
    
    if (activeObject && activeObject.type === 'group') {
      const objects = activeObject.getObjects();
      canvas.remove(activeObject);
      
      objects.forEach(obj => {
        obj.set({
          id: `elem_${Date.now()}_${Math.random()}`,
          left: obj.left + activeObject.left,
          top: obj.top + activeObject.top
        });
        canvas.add(obj);
      });
      
      canvas.renderAll();
      onUngroupElements(activeObject.id);
    }
  }, [onUngroupElements]);

  // Pano işlemleri
  const copySelected = useCallback(() => {
    const canvas = fabricCanvasRef.current;
    const activeObject = canvas.getActiveObject();
    
    if (activeObject) {
      activeObject.clone((cloned) => {
        setClipboard(cloned);
      });
    }
  }, []);

  const pasteClipboard = useCallback(() => {
    if (clipboard) {
      clipboard.clone((clonedObj) => {
        const canvas = fabricCanvasRef.current;
        clonedObj.set({
          left: clonedObj.left + 20,
          top: clonedObj.top + 20,
          id: `elem_${Date.now()}_${Math.random()}`
        });
        
        canvas.add(clonedObj);
        canvas.setActiveObject(clonedObj);
        canvas.renderAll();
        
        const element = convertFabricObjectToElement(clonedObj);
        onElementAdd(element);
      });
    }
  }, [clipboard, onElementAdd]);

  const cutSelected = useCallback(() => {
    copySelected();
    deleteSelected();
  }, []);

  const duplicateSelected = useCallback(() => {
    const canvas = fabricCanvasRef.current;
    const activeObject = canvas.getActiveObject();
    
    if (activeObject) {
      activeObject.clone((cloned) => {
        cloned.set({
          left: cloned.left + 30,
          top: cloned.top + 30,
          id: `elem_${Date.now()}_${Math.random()}`
        });
        
        canvas.add(cloned);
        canvas.setActiveObject(cloned);
        canvas.renderAll();
        
        const element = convertFabricObjectToElement(cloned);
        onElementAdd(element);
      });
    }
  }, [onElementAdd]);

  const deleteSelected = useCallback(() => {
    const canvas = fabricCanvasRef.current;
    const activeObject = canvas.getActiveObject();
    
    if (activeObject) {
      canvas.remove(activeObject);
      canvas.renderAll();
      onElementDelete(activeObject.id);
    }
  }, [onElementDelete]);

  // Geçmiş yönetimi
  const addToHistory = useCallback(() => {
    const canvas = fabricCanvasRef.current;
    const state = canvas.toJSON();
    
    setHistory(prev => {
      const newHistory = prev.slice(0, historyIndex + 1);
      return [...newHistory, state];
    });
    setHistoryIndex(prev => prev + 1);
  }, [historyIndex]);

  const undo = useCallback(() => {
    if (historyIndex > 0) {
      setHistoryIndex(prev => prev - 1);
      loadHistoryState(historyIndex - 1);
    }
  }, [historyIndex]);

  const redo = useCallback(() => {
    if (historyIndex < history.length - 1) {
      setHistoryIndex(prev => prev + 1);
      loadHistoryState(historyIndex + 1);
    }
  }, [historyIndex, history]);

  const loadHistoryState = (index) => {
    const canvas = fabricCanvasRef.current;
    canvas.loadFromJSON(history[index], () => {
      canvas.renderAll();
      canvas.calcOffset();
    });
  };

  // Element dönüştürücü (güncellenmiş)
  const convertFabricObjectToElement = (obj, type = null) => {
    const baseElement = {
      id: obj.id || `elem_${Date.now()}`,
      type: type || obj.type,
      x: obj.left || 0,
      y: obj.top || 0,
      width: obj.width * (obj.scaleX || 1),
      height: obj.height * (obj.scaleY || 1),
      rotation: obj.angle || 0,
      scaleX: obj.scaleX || 1,
      scaleY: obj.scaleY || 1,
      opacity: obj.opacity || 1,
      visible: obj.visible !== false,
      locked: obj.locked || false,
      layer: fabricCanvasRef.current?.getObjects().indexOf(obj) || 0,
      styles: {
        fill: obj.fill,
        stroke: obj.stroke,
        strokeWidth: obj.strokeWidth,
        shadow: obj.shadow,
        gradient: obj.gradient
      }
    };

    // Medya bilgileri
    if (obj.mediaInfo) {
      baseElement.mediaInfo = obj.mediaInfo;
    }
    if (obj.videoInfo) {
      baseElement.mediaInfo = obj.videoInfo;
    }
    if (obj.audioInfo) {
      baseElement.mediaInfo = obj.audioInfo;
    }

    return baseElement;
  };

  return (
    <div className="advanced-slide-canvas">
      <canvas ref={canvasRef} />
      
      <div className="canvas-tools">
        <div className="layer-controls">
          <button onClick={bringToFront} title="En Öne">⏫</button>
          <button onClick={bringForward} title="Öne Getir">⬆️</button>
          <button onClick={sendBackward} title="Geriye Gönder">⬇️</button>
          <button onClick={sendToBack} title="En Arkaya">⏬</button>
        </div>
        
        <div className="group-controls">
          <button onClick={groupSelected} title="Grupla">📁</button>
          <button onClick={ungroupSelected} title="Grubu Çöz">📂</button>
        </div>
        
        <div className="clipboard-controls">
          <button onClick={copySelected} title="Kopyala (Ctrl+C)">📋</button>
          <button onClick={pasteClipboard} title="Yapıştır (Ctrl+V)">📌</button>
          <button onClick={cutSelected} title="Kes (Ctrl+X)">✂️</button>
          <button onClick={duplicateSelected} title="Çoğalt (Ctrl+D)">🔄</button>
        </div>
        
        <div className="history-controls">
          <button onClick={undo} title="Geri Al (Ctrl+Z)">↩️</button>
          <button onClick={redo} title="İleri Al (Ctrl+Y)">↪️</button>
        </div>
      </div>
      
      <div className="canvas-info">
        <span>Katman: {selectedElement?.layer + 1 || '-'}</span>
        <span>Öğe: {selectedElement?.name || 'Seçili değil'}</span>
      </div>
    </div>
  );
};

export default AdvancedSlideCanvas;
3. Medya Yükleme Bileşeni
jsx
// components/Media/MediaUploader.jsx
import React, { useCallback } from 'react';
import { useDropzone } from 'react-dropzone';
import './MediaUploader.css';

const MediaUploader = ({ onMediaAdd, allowedTypes = ['image/*', 'video/*', 'audio/*'] }) => {
  const onDrop = useCallback(async (acceptedFiles) => {
    for (const file of acceptedFiles) {
      const type = file.type.split('/')[0]; // 'image', 'video', 'audio'
      await onMediaAdd(file, type);
    }
  }, [onMediaAdd]);

  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop,
    accept: allowedTypes.reduce((acc, type) => ({ ...acc, [type]: [] }), {})
  });

  return (
    <div className="media-uploader">
      <div {...getRootProps()} className={`dropzone ${isDragActive ? 'active' : ''}`}>
        <input {...getInputProps()} />
        <div className="dropzone-content">
          <span className="upload-icon">📁</span>
          <p>Dosyaları sürükleyin veya tıklayın</p>
          <small>Resim, video veya ses dosyaları</small>
        </div>
      </div>
      
      <div className="media-quick-actions">
        <button className="quick-action" onClick={() => document.getElementById('imageInput').click()}>
          <span>🖼️</span> Resim
        </button>
        <button className="quick-action" onClick={() => document.getElementById('videoInput').click()}>
          <span>🎥</span> Video
        </button>
        <button className="quick-action" onClick={() => document.getElementById('audioInput').click()}>
          <span>🎵</span> Ses
        </button>
      </div>
    </div>
  );
};

export default MediaUploader;
4. Şablon Yöneticisi
jsx
// components/Templates/TemplateManager.jsx
import React, { useState, useEffect } from 'react';
import templateService from '../../services/templateService';
import './TemplateManager.css';

const TemplateManager = ({ onApplyTemplate, onSaveAsTemplate }) => {
  const [templates, setTemplates] = useState([]);
  const [categories, setCategories] = useState([]);
  const [selectedCategory, setSelectedCategory] = useState('all');
  const [searchTerm, setSearchTerm] = useState('');
  const [showSaveDialog, setShowSaveDialog] = useState(false);
  const [newTemplate, setNewTemplate] = useState({
    name: '',
    category: '',
    tags: '',
    description: ''
  });

  useEffect(() => {
    loadTemplates();
    loadCategories();
  }, []);

  const loadTemplates = async () => {
    const data = await templateService.getTemplates();
    setTemplates(data);
  };

  const loadCategories = async () => {
    const cats = await templateService.getCategories();
    setCategories(cats);
  };

  const filteredTemplates = templates.filter(template => {
    const matchesCategory = selectedCategory === 'all' || template.category === selectedCategory;
    const matchesSearch = template.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
                         template.tags?.some(tag => tag.toLowerCase().includes(searchTerm.toLowerCase()));
    return matchesCategory && matchesSearch;
  });

  const handleSaveTemplate = async () => {
    await templateService.saveTemplate({
      ...newTemplate,
      tags: newTemplate.tags.split(',').map(tag => tag.trim()),
      created: new Date()
    });
    setShowSaveDialog(false);
    loadTemplates();
  };

  return (
    <div className="template-manager">
      <div className="template-header">
        <h3>Şablonlar</h3>
        <button onClick={() => setShowSaveDialog(true)} className="save-template-btn">
          💾 Geçerli Slaydı Şablon Kaydet
        </button>
      </div>

      <div className="template-filters">
        <input
          type="text"
          placeholder="Şablon ara..."
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          className="template-search"
        />
        
        <select
          value={selectedCategory}
          onChange={(e) => setSelectedCategory(e.target.value)}
          className="template-category-filter"
        >
          <option value="all">Tüm Kategoriler</option>
          {categories.map(cat => (
            <option key={cat} value={cat}>{cat}</option>
          ))}
        </select>
      </div>

      <div className="template-grid">
        {filteredTemplates.map(template => (
          <div key={template.id} className="template-card">
            <div className="template-preview" style={{ backgroundColor: template.previewColor }}>
              {template.thumbnail ? (
                <img src={template.thumbnail} alt={template.name} />
              ) : (
                <div className="template-placeholder">
                  {template.elements?.length || 0} öğe
                </div>
              )}
            </div>
            
            <div className="template-info">
              <h4>{template.name}</h4>
              <span className="template-category">{template.category}</span>
              <p>{template.description}</p>
              <div className="template-tags">
                {template.tags?.map(tag => (
                  <span key={tag} className="tag">#{tag}</span>
                ))}
              </div>
            </div>
            
            <div className="template-actions">
              <button onClick={() => onApplyTemplate(template)} className="apply-btn">
                Uygula
              </button>
              <button className="favorite-btn">⭐</button>
            </div>
          </div>
        ))}
      </div>

      {showSaveDialog && (
        <div className="modal">
          <div className="modal-content">
            <h3>Şablon Olarak Kaydet</h3>
            
            <input
              type="text"
              placeholder="Şablon adı"
              value={newTemplate.name}
              onChange={(e) => setNewTemplate({ ...newTemplate, name: e.target.value })}
            />
            
            <select
              value={newTemplate.category}
              onChange={(e) => setNewTemplate({ ...newTemplate, category: e.target.value })}
            >
              <option value="">Kategori seçin</option>
              {categories.map(cat => (
                <option key={cat} value={cat}>{cat}</option>
              ))}
            </select>
            
            <input
              type="text"
              placeholder="Etiketler (virgülle ayırın)"
              value={newTemplate.tags}
              onChange={(e) => setNewTemplate({ ...newTemplate, tags: e.target.value })}
            />
            
            <textarea
              placeholder="Açıklama"
              value={newTemplate.description}
              onChange={(e) => setNewTemplate({ ...newTemplate, description: e.target.value })}
            />
            
            <div className="modal-actions">
              <button onClick={handleSaveTemplate} className="save-btn">Kaydet</button>
              <button onClick={() => setShowSaveDialog(false)} className="cancel-btn">İptal</button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
};

export default TemplateManager;
5. Zengin Metin Düzenleyici
jsx
// components/Text/RichTextEditor.jsx
import React, { useState, useEffect } from 'react';
import { Editor, EditorState, RichUtils, convertToRaw, convertFromRaw } from 'draft-js';
import 'draft-js/dist/Draft.css';
import './RichTextEditor.css';

const RichTextEditor = ({ text, onChange, onStyleChange }) => {
  const [editorState, setEditorState] = useState(() => 
    text ? EditorState.createWithContent(convertFromRaw(JSON.parse(text))) : EditorState.createEmpty()
  );

  const handleKeyCommand = (command, editorState) => {
    const newState = RichUtils.handleKeyCommand(editorState, command);
    if (newState) {
      setEditorState(newState);
      return 'handled';
    }
    return 'not-handled';
  };

  const onBoldClick = () => {
    setEditorState(RichUtils.toggleInlineStyle(editorState, 'BOLD'));
  };

  const onItalicClick = () => {
    setEditorState(RichUtils.toggleInlineStyle(editorState, 'ITALIC'));
  };

  const onUnderlineClick = () => {
    setEditorState(RichUtils.toggleInlineStyle(editorState, 'UNDERLINE'));
  };

  const onAlignClick = (alignment) => {
    setEditorState(RichUtils.toggleBlockType(editorState, alignment));
  };

  useEffect(() => {
    const content = editorState.getCurrentContent();
    if (!content.hasText()) return;
    
    const raw = convertToRaw(content);
    onChange(JSON.stringify(raw));
  }, [editorState, onChange]);

  return (
    <div className="rich-text-editor">
      <div className="editor-toolbar">
        <button onClick={onBoldClick} className="toolbar-btn">B</button>
        <button onClick={onItalicClick} className="toolbar-btn">I</button>
        <button onClick={onUnderlineClick} className="toolbar-btn">U</button>
        <button onClick={() => onAlignClick('left')} className="toolbar-btn">⬅️</button>
        <button onClick={() => onAlignClick('center')} className="toolbar-btn">⏺️</button>
        <button onClick={() => onAlignClick('right')} className="toolbar-btn">➡️</button>
        <button onClick={() => onAlignClick('justify')} className="toolbar-btn">↔️</button>
        
        <select onChange={(e) => onStyleChange('fontFamily', e.target.value)}>
          <option value="Arial">Arial</option>
          <option value="Times New Roman">Times New Roman</option>
          <option value="Courier New">Courier New</option>
          <option value="Georgia">Georgia</option>
          <option value="Verdana">Verdana</option>
        </select>
        
        <select onChange={(e) => onStyleChange('fontSize', e.target.value)}>
          <option value="12">12</option>
          <option value="14">14</option>
          <option value="16">16</option>
          <option value="18">18</option>
          <option value="20">20</option>
          <option value="24">24</option>
          <option value="28">28</option>
          <option value="32">32</option>
        </select>
        
        <input 
          type="color" 
          onChange={(e) => onStyleChange('color', e.target.value)}
          title="Metin rengi"
        />
        
        <input 
          type="color" 
          onChange={(e) => onStyleChange('backgroundColor', e.target.value)}
          title="Arka plan rengi"
        />
      </div>
      
      <div className="editor-container">
        <Editor
          editorState={editorState}
          onChange={setEditorState}
          handleKeyCommand={handleKeyCommand}
          placeholder="Metin girin..."
        />
      </div>
    </div>
  );
};

export default RichTextEditor;
6. Şablon Servisi
javascript
// services/templateService.js
class TemplateService {
  constructor() {
    this.storageKey = 'presentation_templates';
    this.apiUrl = process.env.REACT_APP_API_URL || 'http://localhost:8080/api';
  }

  // Yerel depolamadan şablonları getir
  async getTemplates() {
    try {
      const localTemplates = JSON.parse(localStorage.getItem(this.storageKey) || '[]');
      
      // Backend'den de şablonları al
      const response = await fetch(`${this.apiUrl}/templates`);
      const serverTemplates = await response.json();
      
      return [...localTemplates, ...serverTemplates];
    } catch (error) {
      console.error('Şablonlar yüklenirken hata:', error);
      return JSON.parse(localStorage.getItem(this.storageKey) || '[]');
    }
  }

  // Şablon kaydet
  async saveTemplate(template) {
    const templates = await this.getTemplates();
    
    const newTemplate = {
      ...template,
      id: `template_${Date.now()}`,
      created: new Date().toISOString(),
      usageCount: 0
    };
    
    templates.push(newTemplate);
    localStorage.setItem(this.storageKey, JSON.stringify(templates));
    
    // Backend'e de kaydetmeyi dene
    try {
      await fetch(`${this.apiUrl}/templates`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newTemplate)
      });
    } catch (error) {
      console.warn('Backend kaydedilemedi, sadece yerel depolamaya kaydedildi');
    }
    
    return newTemplate;
  }

  // Şablon sil
  async deleteTemplate(templateId) {
    let templates = await this.getTemplates();
    templates = templates.filter(t => t.id !== templateId);
    localStorage.setItem(this.storageKey, JSON.stringify(templates));
    
    try {
      await fetch(`${this.apiUrl}/templates/${templateId}`, {
        method: 'DELETE'
      });
    } catch (error) {
      console.warn('Backend silinemedi, sadece yerel depolamadan silindi');
    }
  }

  // Kategorileri getir
  async getCategories() {
    const templates = await this.getTemplates();
    const categories = [...new Set(templates.map(t => t.category).filter(Boolean))];
    return ['İş', 'Eğitim', 'Pazarlama', 'Yaratıcı', 'Teknik', ...categories];
  }

  // Şablonu kullan (sayacı artır)
  async useTemplate(templateId) {
    const templates = await this.getTemplates();
    const template = templates.find(t => t.id === templateId);
    
    if (template) {
      template.usageCount = (template.usageCount || 0) + 1;
      template.lastUsed = new Date().toISOString();
      localStorage.setItem(this.storageKey, JSON.stringify(templates));
    }
  }

  // Popüler şablonları getir
  async getPopularTemplates(limit = 10) {
    const templates = await this.getTemplates();
    return templates
      .sort((a, b) => (b.usageCount || 0) - (a.usageCount || 0))
      .slice(0, limit);
  }
}

export default new TemplateService();
7. Gelişmiş Editor Bileşeni
jsx
// components/Presentation/AdvancedPresentationEditor.jsx
import React, { useState, useCallback, useRef } from 'react';
import AdvancedSlideCanvas from './AdvancedSlideCanvas';
import MediaUploader from '../Media/MediaUploader';
import TemplateManager from '../Templates/TemplateManager';
import RichTextEditor from '../Text/RichTextEditor';
import { ElementTypes } from '../../types/elementTypes';
import './AdvancedPresentationEditor.css';

const AdvancedPresentationEditor = () => {
  const [slides, setSlides] = useState([]);
  const [currentSlideIndex, setCurrentSlideIndex] = useState(0);
  const [selectedElement, setSelectedElement] = useState(null);
  const [showMediaUploader, setShowMediaUploader] = useState(false);
  const [showTemplateManager, setShowTemplateManager] = useState(false);
  const [showRichTextEditor, setShowRichTextEditor] = useState(false);
  const canvasRef = useRef();

  const currentSlide = slides[currentSlideIndex] || {
    id: `slide_${Date.now()}`,
    elements: [],
    background: '#ffffff'
  };

  // Element ekle
  const handleElementAdd = useCallback((element) => {
    setSlides(prev => {
      const newSlides = [...prev];
      if (!newSlides[currentSlideIndex]) {
        newSlides[currentSlideIndex] = { id: `slide_${Date.now()}`, elements: [] };
      }
      newSlides[currentSlideIndex].elements.push(element);
      return newSlides;
    });
  }, [currentSlideIndex]);

  // Element güncelle
  const handleElementUpdate = useCallback((elementId, updates) => {
    setSlides(prev => {
      const newSlides = [...prev];
      const elements = newSlides[currentSlideIndex].elements;
      const index = elements.findIndex(el => el.id === elementId);
      if (index !== -1) {
        elements[index] = { ...elements[index], ...updates };
      }
      return newSlides;
    });
  }, [currentSlideIndex]);

  // Element sil
  const handleElementDelete = useCallback((elementId) => {
    setSlides(prev => {
      const newSlides = [...prev];
      newSlides[currentSlideIndex].elements = 
        newSlides[currentSlideIndex].elements.filter(el => el.id !== elementId);
      return newSlides;
    });
    setSelectedElement(null);
  }, [currentSlideIndex]);

  // Medya ekle
  const handleMediaAdd = useCallback(async (file, type) => {
    if (canvasRef.current) {
      const element = await canvasRef.current.addMedia(file, type);
      handleElementAdd(element);
      setShowMediaUploader(false);
    }
  }, [handleElementAdd]);

  // Şablon uygula
  const handleApplyTemplate = useCallback((template) => {
    setSlides(prev => {
      const newSlides = [...prev];
      newSlides[currentSlideIndex] = {
        ...template,
        id: `slide_${Date.now()}`,
        elements: template.elements?.map(el => ({
          ...el,
          id: `elem_${Date.now()}_${Math.random()}`
        })) || []
      };
      return newSlides;
    });
    setShowTemplateManager(false);
  }, [currentSlideIndex]);

  // Sunumu kaydet
  const handleSavePresentation = useCallback(async () => {
    const presentationData = {
      id: `pres_${Date.now()}`,
      name: 'Sunum',
      slides: slides,
      created: new Date(),
      modified: new Date()
    };
    
    localStorage.setItem('current_presentation', JSON.stringify(presentationData));
    
    // Backend'e de kaydet
    try {
      const response = await fetch('http://localhost:8080/api/presentation/save', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(presentationData)
      });
      
      if (response.ok) {
        alert('Sunum kaydedildi!');
      }
    } catch (error) {
      console.error('Kaydetme hatası:', error);
      alert('Yerel olarak kaydedildi, ancak sunucuya kaydedilemedi.');
    }
  }, [slides]);

  // Sunumu yükle
  const handleLoadPresentation = useCallback(() => {
    const saved = localStorage.getItem('current_presentation');
    if (saved) {
      const presentation = JSON.parse(saved);
      setSlides(presentation.slides || []);
    }
  }, []);

  return (
    <div className="advanced-presentation-editor">
      <div className="editor-header">
        <h1>Gelişmiş Sunum Editörü</h1>
        <div className="header-actions">
          <button onClick={handleSavePresentation} className="header-btn">
            💾 Kaydet
          </button>
          <button onClick={handleLoadPresentation} className="header-btn">
            📂 Yükle
          </button>
          <button onClick={() => setShowMediaUploader(true)} className="header-btn">
            📁 Medya Ekle
          </button>
          <button onClick={() => setShowTemplateManager(true)} className="header-btn">
            📋 Şablonlar
          </button>
          <button onClick={() => setShowRichTextEditor(true)} className="header-btn">
            ✍️ Metin Düzenle
          </button>
        </div>
      </div>

      <div className="editor-content">
        <div className="slides-panel">
          <h3>Slaytlar</h3>
          <div className="slide-thumbnails">
            {slides.map((slide, index) => (
              <div
                key={slide.id}
                className={`slide-thumbnail ${index === currentSlideIndex ? 'active' : ''}`}
                onClick={() => setCurrentSlideIndex(index)}
              >
                <div className="thumbnail-number">{index + 1}</div>
                <div className="thumbnail-preview">
                  {slide.elements?.length || 0} öğe
                </div>
              </div>
            ))}
            <button 
              className="add-slide-btn"
              onClick={() => {
                setSlides(prev => [...prev, {
                  id: `slide_${Date.now()}`,
                  elements: [],
                  background: '#ffffff'
                }]);
              }}
            >
              + Yeni Slayt
            </button>
          </div>
        </div>

        <div className="canvas-area">
          <AdvancedSlideCanvas
            ref={canvasRef}
            slide={currentSlide}
            selectedElement={selectedElement}
            onElementSelect={setSelectedElement}
            onElementAdd={handleElementAdd}
            onElementUpdate={handleElementUpdate}
            onElementDelete={handleElementDelete}
            onLayerChange={(action, element) => {
              console.log('Katman değişti:', action, element);
            }}
            onGroupElements={(elementIds) => {
              console.log('Gruplandı:', elementIds);
            }}
            onUngroupElements={(groupId) => {
              console.log('Grup çözüldü:', groupId);
            }}
          />
        </div>

        <div className="properties-panel">
          <h3>Özellikler</h3>
          {selectedElement ? (
            <div className="element-properties">
              <div className="property-group">
                <label>Konum</label>
                <div className="property-row">
                  <input
                    type="number"
                    value={Math.round(selectedElement.x)}
                    onChange={(e) => handleElementUpdate(selectedElement.id, { x: parseInt(e.target.value) })}
                    placeholder="X"
                  />
                  <input
                    type="number"
                    value={Math.round(selectedElement.y)}
                    onChange={(e) => handleElementUpdate(selectedElement.id, { y: parseInt(e.target.value) })}
                    placeholder="Y"
                  />
                </div>
              </div>
              
              <div className="property-group">
                <label>Boyut</label>
                <div className="property-row">
                  <input
                    type="number"
                    value={Math.round(selectedElement.width)}
                    onChange={(e) => handleElementUpdate(selectedElement.id, { width: parseInt(e.target.value) })}
                    placeholder="G"
                  />
                  <input
                    type="number"
                    value={Math.round(selectedElement.height)}
                    onChange={(e) => handleElementUpdate(selectedElement.id, { height: parseInt(e.target.value) })}
                    placeholder="Y"
                  />
                </div>
              </div>
              
              <div className="property-group">
                <label>Döndürme</label>
                <input
                  type="range"
                  min="0"
                  max="360"
                  value={selectedElement.rotation || 0}
                  onChange={(e) => handleElementUpdate(selectedElement.id, { rotation: parseInt(e.target.value) })}
                />
              </div>
              
              <div className="property-group">
                <label>Opaklık</label>
                <input
                  type="range"
                  min="0"
                  max="1"
                  step="0.1"
                  value={selectedElement.opacity || 1}
                  onChange={(e) => handleElementUpdate(selectedElement.id, { opacity: parseFloat(e.target.value) })}
                />
              </div>
              
              {selectedElement.type === ElementTypes.TEXT && (
                <div className="property-group">
                  <label>Metin Stili</label>
                  <button onClick={() => setShowRichTextEditor(true)}>
                    Metni Düzenle
                  </button>
                </div>
              )}
              
              {selectedElement.mediaInfo && (
                <div className="property-group">
                  <label>Medya Bilgileri</label>
                  <div className="media-info">
                    <p>Tip: {selectedElement.mediaInfo.type}</p>
                    <p>Boyut: {Math.round(selectedElement.mediaInfo.size / 1024)} KB</p>
                  </div>
                </div>
              )}
            </div>
          ) : (
            <p>Bir öğe seçin</p>
          )}
        </div>
      </div>

      {showMediaUploader && (
        <div className="modal">
          <div className="modal-content">
            <button className="close-btn" onClick={() => setShowMediaUploader(false)}>✖</button>
            <MediaUploader onMediaAdd={handleMediaAdd} />
          </div>
        </div>
      )}

      {showTemplateManager && (
        <div className="modal large">
          <div className="modal-content">
            <button className="close-btn" onClick={() => setShowTemplateManager(false)}>✖</button>
            <TemplateManager
              onApplyTemplate={handleApplyTemplate}
              onSaveAsTemplate={(template) => {
                templateService.saveTemplate({
                  ...template,
                  elements: currentSlide.elements
                });
              }}
            />
          </div>
        </div>
      )}

      {showRichTextEditor && (
        <div className="modal">
          <div className="modal-content">
            <button className="close-btn" onClick={() => setShowRichTextEditor(false)}>✖</button>
            <RichTextEditor
              text={selectedElement?.content}
              onChange={(content) => {
                if (selectedElement) {
                  handleElementUpdate(selectedElement.id, { content });
                }
              }}
              onStyleChange={(property, value) => {
                if (selectedElement) {
                  handleElementUpdate(selectedElement.id, {
                    styles: { ...selectedElement.styles, [property]: value }
                  });
                }
              }}
            />
          </div>
        </div>
      )}
    </div>
  );
};

export default AdvancedPresentationEditor;
8. CSS Stilleri
css
/* AdvancedPresentationEditor.css */
.advanced-presentation-editor {
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #f5f5f5;
}

.editor-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 2rem;
  background: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.editor-header h1 {
  margin: 0;
  font-size: 1.5rem;
  color: #333;
}

.header-actions {
  display: flex;
  gap: 0.5rem;
}

.header-btn {
  padding: 0.5rem 1rem;
  border: 1px solid #ddd;
  background: white;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  transition: all 0.2s;
}

.header-btn:hover {
  background: #f0f0f0;
  border-color: #999;
}

.editor-content {
  flex: 1;
  display: flex;
  overflow: hidden;
}

.slides-panel {
  width: 200px;
  background: white;
  border-right: 1px solid #ddd;
  display: flex;
  flex-direction: column;
  padding: 1rem;
}

.slides-panel h3 {
  margin: 0 0 1rem 0;
  font-size: 1rem;
  color: #666;
}

.slide-thumbnails {
  flex: 1;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.slide-thumbnail {
  position: relative;
  height: 100px;
  background: #f9f9f9;
  border: 2px solid transparent;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
}

.slide-thumbnail.active {
  border-color: #2196f3;
  background: #e3f2fd;
}

.slide-thumbnail:hover {
  background: #f0f0f0;
}

.thumbnail-number {
  position: absolute;
  top: 5px;
  left: 5px;
  background: rgba(0,0,0,0.6);
  color: white;
  padding: 2px 6px;
  border-radius: 10px;
  font-size: 10px;
}

.thumbnail-preview {
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  color: #999;
}

.add-slide-btn {
  padding: 0.5rem;
  background: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 1rem;
}

.canvas-area {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 2rem;
  background: #e0e0e0;
  overflow: auto;
}

.properties-panel {
  width: 300px;
  background: white;
  border-left: 1px solid #ddd;
  padding: 1rem;
  overflow-y: auto;
}

.properties-panel h3 {
  margin: 0 0 1rem 0;
  font-size: 1rem;
  color: #666;
}

.element-properties {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.property-group {
  border: 1px solid #eee;
  border-radius: 4px;
  padding: 0.5rem;
}

.property-group label {
  display: block;
  font-size: 12px;
  color: #999;
  margin-bottom: 0.5rem;
}

.property-row {
  display: flex;
  gap: 0.5rem;
}

.property-row input {
  flex: 1;
  padding: 0.25rem;
  border: 1px solid #ddd;
  border-radius: 2px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  max-width: 800px;
  max-height: 80vh;
  overflow-y: auto;
  position: relative;
}

.modal.large .modal-content {
  max-width: 1200px;
  width: 90%;
}

.close-btn {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: #999;
}

.close-btn:hover {
  color: #333;
}

/* AdvancedSlideCanvas.css */
.advanced-slide-canvas {
  position: relative;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
  background: white;
}

.canvas-tools {
  position: absolute;
  top: 10px;
  left: 10px;
  display: flex;
  gap: 5px;
  background: rgba(255,255,255,0.9);
  padding: 5px;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  z-index: 100;
}

.canvas-tools button {
  width: 32px;
  height: 32px;
  border: 1px solid #ddd;
  background: white;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
}

.canvas-tools button:hover {
  background: #f0f0f0;
}

.canvas-info {
  position: absolute;
  bottom: 10px;
  right: 10px;
  background: rgba(0,0,0,0.6);
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  z-index: 100;
}

.canvas-info span {
  margin-left: 10px;
}
Bu gelişmiş yapı ile:

✅ Katman yönetimi: Öne/arkaya alma, gruplama, kilitlieme
✅ Medya desteği: Resim, video, ses dosyaları ekleme
✅ Şablon sistemi: Slaytları şablon olarak kaydedip tekrar kullanma
✅ Zengin metin düzenleyici: Gelişmiş metin biçimlendirme
✅ Pano işlemleri: Kopyala, yapıştır, kes, çoğalt
✅ Geçmiş yönetimi: Geri/ileri al
✅ Klavye kısayolları: Profesyonel kullanım için
✅ Medya bilgileri: Dosya boyutu, tip vb.
✅ Yerel ve sunucu depolama: Şablonları ve sunumları kaydetme

İhtiyacınız olan ek özellikler varsa söyleyin, onları da ekleyelim! 🚀