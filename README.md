# Procédure de Mise en Œuvre de l’Outil JasperReports avec Spring Boot

## 1. Configuration Initiale

### 1.1 Ajouter les dépendances Maven

Ajoutez les dépendances suivantes à votre fichier `pom.xml` :

```xml
<dependencies>

    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- JasperReports -->
    <dependency>
        <groupId>net.sf.jasperreports</groupId>
        <artifactId>jasperreports</artifactId>
        <version>6.20.0</version>
    </dependency>

    <!-- Pour l'export en PDF -->
    <dependency>
        <groupId>org.apache.pdfbox</groupId>
        <artifactId>pdfbox</artifactId>
        <version>2.0.24</version>
    </dependency>

    <!-- Pour l'export en Excel -->
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi</artifactId>
        <version>5.2.2</version>
    </dependency>

</dependencies>
```

### 1.2 Structure du projet 
```
src/
├── main/
│   ├── java/
│   │   └── com/example/jasperdemo/
│   │       ├── config/
│   │       ├── controller/
│   │       ├── model/
│   │       ├── repository/
│   │       ├── service/
│   │       └── JasperDemoApplication.java
│   └── resources/
│       ├── reports/          # Dossier pour les fichiers .jrxml
│       ├── static/           # Fichiers statiques
│       └── templates/        # Fichiers de template
```
---

## 2. Création d’un Rapport Simple

### 2.1 Créer un fichier JRXML

Dans le dossier `src/main/resources/reports/`, créez un fichier nommé `simple_report.jrxml`.

> 💡 Ce fichier contient la structure du rapport au format XML généré par Jaspersoft Studio. Vous pouvez y définir des titres, colonnes, champs, etc.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jasperReport xmlns="http://jasperreports.sourceforge.net/jasperreports" 
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
              xsi:schemaLocation="http://jasperreports.sourceforge.net/jasperreports http://jasperreports.sourceforge.net/xsd/jasperreport.xsd" 
              name="simple_report" 
              pageWidth="595" 
              pageHeight="842" 
              columnWidth="555" 
              leftMargin="20" 
              rightMargin="20" 
              topMargin="20" 
              bottomMargin="20">
    <title>
        <band height="50">
            <staticText>
                <reportElement x="0" y="0" width="555" height="30"/>
                <text><![CDATA[Hello JasperReports from Spring Boot!]]></text>
            </staticText>
        </band>
    </title>
</jasperReport>
```

---

## 3. Service de Génération de Rapports

### 3.1 Créer un service `JasperReportService`

```java
package com.example.jasperdemo.service;

import net.sf.jasperreports.engine.*;
import net.sf.jasperreports.engine.data.JRBeanCollectionDataSource;
import org.springframework.stereotype.Service;
import org.springframework.util.ResourceUtils;

import java.io.File;
import java.io.FileNotFoundException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Service
public class JasperReportService {

    public byte[] generateSimpleReport() throws FileNotFoundException, JRException {
        // Chargement du fichier JRXML
        File file = ResourceUtils.getFile("classpath:reports/simple_report.jrxml");
        JasperReport jasperReport = JasperCompileManager.compileReport(file.getAbsolutePath());

        // Paramètres du rapport
        Map<String, Object> parameters = new HashMap<>();
        parameters.put("createdBy", "Spring Boot");

        // Source de données (vide dans cet exemple)
        JRBeanCollectionDataSource dataSource = new JRBeanCollectionDataSource(List.of());

        // Remplissage du rapport
        JasperPrint jasperPrint = JasperFillManager.fillReport(jasperReport, parameters, dataSource);

        // Export au format PDF
        return JasperExportManager.exportReportToPdf(jasperPrint);
    }
}
```
## 4. Controller pour exposer l’API

### 4.1 Créer un controller `ReportController`

Ce contrôleur expose une route HTTP GET permettant de générer et télécharger le rapport PDF.

```java
package com.example.jasperdemo.controller;

import com.example.jasperdemo.service.JasperReportService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/reports")
public class ReportController {

    @Autowired
    private JasperReportService jasperReportService;

    @GetMapping("/simple")
    public ResponseEntity<byte[]> generateSimpleReport() {
        try {
            byte[] reportContent = jasperReportService.generateSimpleReport();

            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_PDF);
            headers.setContentDispositionFormData("filename", "simple_report.pdf");
            headers.setCacheControl("must-revalidate, post-check=0, pre-check=0");

            return ResponseEntity.ok()
                    .headers(headers)
                    .body(reportContent);

        } catch (Exception e) {
            return ResponseEntity.internalServerError().build();
        }
    }
}
```
> 📎 Résultat attendu : Un appel HTTP à GET /api/reports/simple renvoie le fichier PDF généré contenant le rapport.





