<!SLIDE subsection>

# Integrating Spring and JavaServer Faces #
<br><br>
## Phillip Webb ##
### _SpringSource, VMware_ ###

<!SLIDE>

# Overview #

<!SLIDE>

# The demo application #

<!SLIDE subsection>

# Spring in control #

<!SLIDE bullets incremental>

# How? #

* JSF becomes the 'V' in MVC
* <tt>@Controller</tt> and <tt>@RequestMapping</tt> like any other Spring MVC
* Put your XHTML in <tt>/WEB-INF/pages/</tt>
* Use all Spring goodness (<tt>@PathVariable</tt>, <tt>@RequestParam</tt> etc)

<!SLIDE bullets incremental>

# Why? #

* Easy to use MVC programming model
* Powerful JSF component libraries
* Simple to mix JSF with other view technologies
* AJAX out the box


<!SLIDE small>

# Example #

	@@@ java
	@Controller
	public class CityController {

		@RequestMapping("/advisor/{country}/{name}")
		public String showCity(
				@PathVariable String country, 
				@PathVariable String name, 
				Model model) {

			model.addAttribute(
				cityService.getCity(name, country));

			return "city";
		}
	}

<!SLIDE small>

# /WEB-INF/pages/city.xhtml #

	@@@ xml
	<!DOCTYPE html 
		PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
			"http://www.w3.org/TR/xhtml1/DTD/xhtm...">

	<html xmlns="http://www.w3.org/1999/xhtml"
		xmlns:h="http://java.sun.com/jsf/html"
		xmlns:f="http://java.sun.com/jsf/core">

		<!-- Page content -->

	</html>


<!SLIDE subsection>

# Reaching the Beans #

<!SLIDE small>

# Spring Beans available using EL #

	@@@ java
	@Component	
	public class ReviewUtils {
		public int getRating(Review review) {
			return review.getRating().ordinal();
		}
	}

<p/>
	@@@ xml
	value="#{reviewUtils.getRating(rating)}"

<!SLIDE small>

# Model as well #

	@@@ java
	@RequestMapping("/advisor/cities/search")
	public String enterSearchDetails(Model model) {
		model.addAttribute(new CitySearchCriteria());
		return "citysearch";
	}
<p/>
	@@@ xml
	value="#{citySearchCriteria.name}"

<!SLIDE bullets incremental>

# Special implicit variables #

* <tt>controller</tt> - The MVC controller
* <tt>handler</tt> - The MVC handler
* <tt>model</tt> - The model map

<!SLIDE small>

# Example: #

	@@@ java
	@Controller
	public class CityController {
		public String getTime() {
			return new Date();
		}
	}
<p/>
	@@@ xml
	value="#{controller.time}"

<!SLIDE subsection>

# Demo #

<!SLIDE subsection>

# Navigation #

<!SLIDE bullets incremental>

# Implicit JSF navigation #

* Use <tt>spring:</tt> prefix with JSF implicit navigation
* Reference any MVC view
* Same as String return from <tt>@RequestMapping</tt>
* Must have JSF implicit views enabled

<!SLIDE small>

# Example: #

	@@@ xml
	<h:commandButton value="Go" 
		action="spring:
		  redirect:http://www.springsource.org"/>

<!SLIDE bullets incremental>

# Navigate to <tt>@RequestMapping</tt> #

* Special <tt>'@'</tt> prefix redirects to other <tt>@RequestMapping</tt>
* URL is built for you
* Refer to the current controller <tt>'@performSearch'</tt>
* Or other controller <tt>'@cityController.performSearch'</tt>

<!SLIDE small>

# Example: #

	@@@ java
	@RequestMapping("/advisor/cities/search")
	public String enterSearchDetails(Model model) {
		model.addAttribute(new CitySearchCriteria());
		return "citysearch";
	}
<p/>
	@@@ xml
	<h:commandButton value="Go" 
		action="spring:@enterSearchDetails"/>

## Redirects to: __<tt>/app/spring/advisor/cities/search</tt>__ ##

<!SLIDE bullets incremental>

# Complex URLs #

* Both <tt>@RequestParam</tt> and <tt>@PathVariable</tt> respected
* Model objects picked by type
* Or use <tt>f:param</tt> tags
* Works with <tt>h:link</tt> to create bookmarkable URLs

<!SLIDE small>

# Example: #
	
	@@@ java
	@RequestMapping("/advisor/cities")
	public String performSearch(
		CitySearchCriteria searchCriteria) {
		
		String name = searchCriteria.getName();
		int resultsPerPage = searchCriteria.getPerPage();
		...

<p/>
	@@@ xml
	<h:commandButton value="Go" 
		action="spring:@performSearch"/>

## Redirects to: __<tt>/app/spring/advisor/cities?name=vegas&perPage=10</tt>__ ##


<!SLIDE small>

# Example: #

	@@@ java
	@RequestMapping("/advisor/{country}/{city}/{name}")
	public String hotel(@PathVariable String country, 
		@PathVariable String city, @PathVariable String name) {
		...
<p/>
	@@@ xml
	<h:link value="#{hotel.name}" 
			outcome="spring:@hotelController.hotel">
	 <f:param name="country" value="#{hotel.city.country}"/>
	 <f:param name="city" value="#{hotel.city.name}"/>
	 <f:param name="name" value="#{hotel.name}"/>
	</h:link>

## Links to: __<tt>/app/spring/advisor/uk/bath/travelodge</tt>__ ##

<!SLIDE bullets incremental>

# Programmatic Navigation #

* Use <tt>@NavigationMapping</tt> in your controller
* Name of the method is the JSF 'action'
* Model arguments matched based on type
* Use <tt>@Value('#{expression}')</tt> arguments to access JSF objects
* Use <tt>NavigationContext</tt> argument for context information

<!SLIDE small>

# Example: #

	@@@ java
	@NavigationMapping
	public Object onSearch(CitySearchCriteria criteria) {

		City city = findSingleCity(criteria);
		
		if (city != null) {
			ExtendedModelMap m = new ExtendedModelMap()
			  .addAttribute("country", city.getCountry())
			  .addAttribute("name", city.getName());
			return new NavigationOutcome("@showCity", m);
		}

		return "@performSearch";
	}

<!SLIDE subsection>

# Demo #

<!SLIDE subsection>

# Select Items #

<!SLIDE bullets incremental>

# Select Items #

* Use in JSF for lists, radio buttons and combo boxes
* Drop-in replacements for <tt>f:selectItems</tt>
* No <tt>f:converter</tt> required
* JPA <tt>@ID</tt> used 
* Quickly add 'Please Select' option

<!SLIDE small>

# Example: #

	@@@ xml
	<p:selectOneMenu value="#{model.selectedCity}" 
			effect="fade" required="true">
		<s:selectItems value="#{cities}"/>
	</p:selectOneMenu>
<p/>

## Produces the following HTML: ##

	@@@ html
	<select>
		<option selected="selected" 
			value="">--- Please Select ---</option>
		<option value="16">Miami</option>
		<option value="17">Melbourne</option>
	</select>

<!SLIDE bullets incremental>

# Deduced Select Items #

* <tt>Enum</tt> and <tt>Boolean</tt> items can be deduced
* <tt>Boolean</tt> gives <tt>'Yes'</tt>, <tt>'No'</tt> options
* <tt>Enum</tt> uses <tt>Enum.values()</tt>
* No <tt>s:selectItems</tt> value is required

<!SLIDE small>

# Example: #

	@@@ xml
	<p:selectOneMenu label="Trip Type" 
			value="#{review.tripType}" effect="fade">
		<s:selectItems/>
	</p:selectOneMenu>

## Produces the following HTML: ##

	@@@ html
	<select>
		<option selected="selected" value="">
				--- Please Select ---</option>
		<option value="BUSINESS">Business</option>
		<option value="COUPLES">Couples</option>
		<option value="FAMILY">Family</option>
		<option value="FRIENDS">Friends</option>
		<option value="SOLO">Solo</option>
	</select>

<!SLIDE subsection>

# Demo #

<!SLIDE subsection>

# Pagination #

<!SLIDE bullets incremental>

# JSF Pagination #

* Easy to get wrong
* Don't fetch all results then display a page
* Use JPA <tt>setFirstResult()</tt> and <tt>setMaxResult()</tt>
* UIData <tt>setRows()</tt> and <tt>setFirst()</tt> not enough

<!SLIDE bullets incremental>

# Using <tt><s:pagedData></tt> #

* Creates lazy loading paged <tt>DataModel</tt>
* Part of your page mark-up
* WARNING: Not quite MVC...
* ... but easy to use

<!SLIDE small>

# Example: #

Expose paged data under <tt>#{myDataModel}</tt>

	@@@ xml
	<s:pagedData
		var="myDataModel" pageSize="20"






	 />

<!SLIDE small>

# Example: #

Obtain a page of data calling <tt>userRepository.findByLastName</tt>

	@@@ xml
	<s:pagedData
		var="myDataModel" pageSize="20"
		value="#{userRepository.findByLastName(
	    	backingBean.lastName, 
			...
			..."


	 />
 
<!SLIDE small>

# Example: #

Details of the page to return available in <tt>pageRequest</tt>

	@@@ xml
	<s:pagedData
		var="myDataModel" pageSize="20"
		value="#{userRepository.findByLastName(
	    	backingBean.lastName, 
	    	pageRequest.offset, 
	    	pageRequest.pageSize)}"


	 />
 
<!SLIDE small>

# Example: #

Call <tt>userRepository.countByLastName</tt> for the total row count 

	@@@ xml
	<s:pagedData
		var="myDataModel" pageSize="20"
		value="#{userRepository.findByLastName(
	    	backingBean.lastName, 
	    	pageRequest.offset, 
	    	pageRequest.pageSize)}"
		rowCount="#{userRepository.countByLastName(
	  		backingBean.lastName)}" 
	 />
 

<!SLIDE small>

# Example: #

	@@@ xml
	<t:dataTable 
			value="#{myDataModel}" 
			rows="#{myDataModel.pageSize}" 
    		var="item">
  		<t:column>
			<h:outputText value="#{item.name}" />
  		</t:column>
  		<t:dataScroller paginator="true" 
  				paginatorMaxPages="9" />
	</t:dataTable>

<!SLIDE bullets incremental>

# Spring Data #

* Pagination and Sorting out the box
* <tt>#{pageRequest}</tt> implements <tt>Pageable</tt> Interface
* <tt>PagedDataModel</tt> has <tt>setSortColumn()</tt> and <tt>toggleSort()</tt>
* Optional dependency, degrades gracefully

<!SLIDE bullets incremental>

# PrimeFaces #

* Use PrimeFaces <tt>p:dataTable</tt> to quickly add paginator
* <tt>#{dataModel}</tt> extends PrimeFaces <tt>LazyDataModel</tt>
* Again, optional and degrades gracefully


<!SLIDE small>

# Example: #

	@@@ java
	Page<HotelSummary> getHotels(City city, 
		Pageable pageable);

<p/>

	@@@ xml
	<s:pagedData value="#{cityService.getHotels(
			city, pageRequest)}" 
		sortColumn="averageRating" 
		sortAscending="#{false}" 
		var="hotels"/>

<!SLIDE small>

# Example: #
	
	@@@ xml
	<p:dataTable value="#{hotels}" paginator="true" 
		rows="#{hotels.pageSize}" var="hotel">
			<p:column headerText="Average Rating" 
				sortBy="#{hotel.averageRating}">
					<h:outputText 
						value="#{hotel.averageRating}"/>
			</p:column>
	</p:dataTable>

<!SLIDE subsection>

# Demo #

<!SLIDE subsection>

# Templates #

<!SLIDE bullets incremental>

# JSF Templates #

* Good support since JSF 2.0
* Use <tt><ui:composition></tt> and <tt><ui:decorate></tt>
* Can get repetitive, especially for forms

<!SLIDE small>

# Example: #

	@@@ xml
	<ui:composition>
		<div class="formElement">
			<span class="formLabel">
				<h:outputLabel for="#{for}" 
					label="#{label}
					  #{showAsterisk ? ' *' : ''}">
			</span>
			<ui:insert/>
		</div>
	</ui:composition>


<!SLIDE small>

# Example: #

	@@@ xml
	<ui:decorate template="/WEB-INF/layout/form.xhtml">
		<h:inputText id="firstName" label="First Name" 
			value="#{bean.firstName}" required="false"/>
		<ui:param name="label" value="First Name"/>
		<ui:param name="for" value="firstName"/>
		<ui:param name="showAsterisk" value="true"/>		
	</ui:decorate>

	<ui:decorate template="/WEB-INF/layout/form.xhtml">
		<h:inputText id="lastName" label="Last Name" 
			value="#{bean.lastName}" required="true"/>
		<ui:param name="label" value="Last Name"/>
		<ui:param name="for" value="lastName"/>
		<ui:param name="showAsterisk" value="true"/>		
	</ui:decorate>
	<!-- More form elements -->

<!SLIDE bullets incremental>

# Component Info #

* Use <tt><s:componentInfo></tt> to introspect component details
* <tt>isValid()</tt>, <tt>isRequired()</tt>, <tt>getLabel()</tt>, <tt>getFor()</tt>
* Child is used so no need to reference component by ID


<!SLIDE small>

# Example: #

	@@@ xml
	<ui:composition>
		<s:componentInfo var="info">
			<div class="formElement">
				<span class="#{info.valid ? 'formLabel' : 
						'formErrorLabel'}">
					<h:outputLabel for="#{info.for}" 
						label="#{info.label}
						  #{info.required ? ' *' : ''}">
				</span>
				<ui:insert/>
			</div>
		</s:componentInfo>
	</ui:composition>


<!SLIDE small>

# Example: #

	@@@ xml
	<ui:decorate template="/WEB-INF/layout/form.xhtml">
		<h:inputText id="firstName" label="First Name" 
			value="#{bean.firstName}" required="false"/>
	</ui:decorate>

	<ui:decorate template="/WEB-INF/layout/form.xhtml">
		<h:inputText id="lastName" label="Last Name" 
			value="#{bean.lastName}" required="true"/>
	</ui:decorate>
	<!-- More form elements -->

<!SLIDE bullets incremental>

# Decorate All #

* Use <tt><s:decorateAll></tt> apply template to each child component
* Still supports <tt><ui:param/></tt> and <tt><ui:define/></tt> tags
* Below the component to apply just once
* Or at the very top to apply to all

<!SLIDE small>

# Example: #

	@@@ xml
	<s:decorateAll template="/WEB-INF/layout/form.xhtml">

		<h:inputText id="firstName" label="First Name" 
			value="#{bean.firstName}" required="false"/>
	
		<h:inputText id="lastName" label="Last Name" 
			value="#{bean.lastName}" required="true"/>
	
		<!-- More form elements -->
	</s:decorateAll>

<!SLIDE subsection>

# Demo #

<!SLIDE subsection>

# Internationalization #

<!SLIDE bullets incremental>

# Message Source #

* Use <tt><s:messageSource></tt> instead of <tt><f:loadBundle></tt>
* Works with Spring <tt>MessageSource</tt>
* Message keys includes page ID to prevent name clashes
* Shortcuts to resolve parameters

<!SLIDE small>

# Example: #

	@@@ properties
	pages.message.simple.welcome=
		Welcome to {1} with {0}
<p/>
	@@@ xml
	<h:outputFormat value="#{messages.welcome}">
		<f:param value="Spring"/>
		<f:param value="JSF"/>
	</h:outputFormat>

<!SLIDE small>

# Example: #

	@@@ properties
	pages.message.simple.welcome=
		Welcome to {1} with {0}
<p/>
	@@@ xml
	<h:outputText 
	  value="#{messages.welcome['Spring']['JSF']}"/>

<!SLIDE bullets incremental>

# Object Message Source #

* New <tt>ObjectMessageSource</tt> extends Spring <tt>MessageSource</tt>
		
*		@@@ java
		String getMessage(Object object, 
			Object[] args, Locale locale) 

<!SLIDE small>

# Example: #

	@@@ java
	package org.example;
	public class ExampleObject {
	}
<p/>
	@@@ xml
	<h:outputText 
		value="#{messages[exampleInstance]}"/>
<p/>
	@@@ properties
	org.example.ExampleObject=example

<!SLIDE small>

	@@@ java
	package org.springframework.springfaces.
					traveladvisor.domain;
	public enum TripType {
		BUSINESS, 
		COUPLES
	}
<p/>
	@@@ xml
	<p:selectOneMenu label="Trip Type" 
			value="#{review.tripType}" effect="fade">
		<s:selectItems/>
	</p:selectOneMenu>
<p/>
	@@@ properties
	org.springframework.springfaces.traveladvisor.
		domain.TripType.BUSINESS=Business
	org.springframework.springfaces.traveladvisor.
		domain.TripType.COUPLES=Couples

<!SLIDE small>

	@@@ java
	package org.example;
	public class PersonName {
		public String getFirst() {
		}

		public String getLast() {
		}
	}
<p/>
	@@@ properties
	org.example.PersonName=Name is {first} {last}

<!SLIDE subsection>

# Loose Ends #

<!SLIDE bullets incremental>

# JSF Converters #

* Spring Beans can be JSF Converters
* ID of the bean is the name of the converter
* Use <tt>@ForClass</tt> annotation to register for a class
* Generic variant of <tt>Converter</tt> also available

<!SLIDE small>

	@@@ java
	@Component
	@ForClass
	public class ExampleConverter implements 
							Converter<MyClass> {

	public MyClass getAsObject(FacesContext context, 
			UIComponent component, String value) {
		// ...
	}

	public String getAsString(FacesContext context, 
			UIComponent component, MyClass value) {
		// ...
	}

<!SLIDE bullets incremental>

# MVC Converters #

* JSF Converters will be used in MVC <tt>@Controller</tt>
* Converters are located from the argument class
* Or use <tt>@FacesConverterId</tt>

<!SLIDE small>

	@@@ java
	@Controller
	public class ConverterExampleController {
		
		@RequestMapping("/converter/facesbyid")
		public Model facesById(
			@RequestParam 
			@FacesConverterId("byId") 
			ConvertedObject value) {
		}
	}

<!SLIDE bullets incremental>

# Validators #

* Spring Beans can be JSF Validators
* ID of the bean is the name of the validator
* Use <tt>@ForClass</tt> annotation to register for a class
* Generic variant of <tt>Validator</tt> also available

<!SLIDE small>

	@@@ java
	@Component
	@ForClass
	public class ExampleValidator implements 
							Validator<MyClass> {
		
		public void validate(FacesContext context, 
				UIComponent component, MyClass value) 
				throws ValidatorException {
			// ...
		}
	}

<!SLIDE bullets incremental>

# Exception Handling #

* Spring beans can handle JSF Exceptions
* Use <tt>ExceptionHandler</tt> interface instead of JSF Class
* MVC <tt>@ExceptionHandler</tt> can also be used
* Exception messages can be displayed using <tt>ObjectMessageSource</tt>

<!SLIDE small>

	@@@ java
	@Component
	@Order(100)
	public class MyExceptionHandler implements 
					ExceptionHandler<MyException> {
		
		boolean handle(MyException exception, 
				  ExceptionQueuedEvent event) 
				  throws Exception {

			// if handled return true

		}
	}

<!SLIDE small>

	@@@ java
	@Controller
	public class ExampleController {
		
		// ... 

		@ExceptionHandler
		public String handleFacesView(MyException e) {
			return "redirect:error";
		}
	}


<!SLIDE small>

	@@@ properties
	# messages_en.properties
	com.corp.product.exception.MyException=
		My Localized Exception Message

<!SLIDE subsection>

# Summary #

<!SLIDE bullets incremental>

# Summary #

* Get really deep JSF/Spring integration
* Flexibility of Spring MVC
* Convenience of JSF Libraries
* Apache 2.0 Licensed

<!SLIDE subsection>

# Thank You #

http://github.com/philwebb/springfaces/
