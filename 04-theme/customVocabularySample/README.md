# customVocabularySample

This folder contains sample custom vocabularies for use with [Omeka-S](https://omeka.org/s/), designed to help archivists and librarians enhance metadata precision and consistency in their digital collections.

**Watch the step-by-step tutorial:**  
[**Omeka S Vocabularies – Vimeo Video**](https://vimeo.com/449764902)

## Purpose

Archivists often need vocabularies tailored to specific institutions, communities, metadata properties or collection types. This folder provides example files as a starting point to help users define their own controlled vocabularies in Omeka-S using the proper structure and format.

By using these examples as a guide, archivists can create their own turtle(.ttl) vocabularies to support custom metadata fields, controlled term lists, or local cataloging practices.

---
## Step 1: Define Your Namespace and URI

### Each vocabulary must begin with a unique namespace URI. Here's how to create one: 
   1. Choose a base domain – Use a domain you control (e.g., [https://yourinstitution.edu/omeka/vocab/](https://yourinstitution.edu/omeka/vocab/)). 
   2. Create a namespace identifier – Choose a short, unique name for your vocabulary (e.g., yourCollection). 
   3. Create a static HTML file: (e.g., [https://yourinstitution.edu/omeka/vocab/yourCollection](https://yourinstitution.edu/omeka/vocab/yourCollection))

This can be a simple HTML page that explains what the vocabulary is. For example: 
```
<!DOCTYPE html> 
<html> 
<head><title>YourCollection Vocabulary</title></head> 
<body> 
  <h1>YourCollection Vocabulary</h1> 
  <p>This namespace defines custom properties used in the YourCollection archival project at your University.</p> 
</body> 
</html> 
  ```
   4. Please review the “GlobalVocabulary.html” file for a full example.
   5. Ensure the URL resolves in a browser, even if it’s a static page. Omeka S does not parse the HTML, it just requires a valid, resolvable URI for semantic integrity. 
   
## Customizing Your Vocabulary

The .ttl sample file follows the [turtle](https://www.w3.org/TR/turtle/) format. You can:
- Edit or replace term labels and URIs.
- Add or remove fields relevant to your collection.
- Include descriptions, comments, or examples for each term.

## Step 2: Customize the provided Turtle (.ttl) Vocabulary File 

1. Open the sample Turtle file provided by the project team. Review the file’s structure, prefixes, and property definitions. 
2. Edit the sample to reflect your collection’s needs: 
    a. Update the namespace URI to match your institution’s vocabulary location. 
    b. Replace sample properties (e.g., Identifier, CreatedBy, State, County) with terms relevant to your archive. 
    c. Add or remove terms, labels, comments, and expected data types as needed. 

Example structure: 

 <pre>```
@prefix rfc: <https://yourinstitution.edu/omeka/vocab/yourcollection#> . 
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> . 
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> . 
 
<https://yourinstitution.edu/omeka/vocab/yourcollection> 
    a rdf:Property ; 
    rdfs:label "Your Collection Vocabulary" ; 
    rdfs:comment "Custom vocabulary for describing items in Your Collection." . 
 
your:Identifier a rdf:Property ; 
    rdfs:label "Identifier" ; 
    rdfs:comment "A unique identifier for the item." ; 
    rdfs:domain rdf:Resource ; 
    rdfs:range xsd:string . 
 
your:CreatedBy a rdf:Property ; 
    rdfs:label "Created By" ; 
    rdfs:comment "The person or organization that created the item." ; 
    rdfs:domain rdf:Resource ; 
    rdfs:range xsd:string . 
</pre>

We recommend validating any modified files before importing them into your Omeka-S instance.


---

## Upload the Vocabulary in Omeka-S

1. In your Omeka S admin panel, go to Admin > Vocabularies. 
2. Click “Import”. 
3. Label the vocabulary 
4. Add your Namespace URI 
5. Add a Namespace prefix 
6. Upload the .ttl file you just created. 
7. Click “Import” to complete the installation. 
8. Omeka will parse and register your vocabulary under your namespace prefix. 

**Watch the step-by-step tutorial:**  
[**Omeka S Vocabularies – Vimeo Video**](https://vimeo.com/449764902)

---




## Notes & Best Practices 

* Custom HTML URI: Even though using an RDF document would be more semantically linked-data-compliant, using an HTML approach is fine for internal vocabularies. The key is that the URI resolves. 
* URI Stability: Never change the namespace URI once in use. Items referencing these properties will break if the URI changes. 
* Turtle Validation: Before uploading, you can validate your Turtle file using an online validator of your choosing. 


