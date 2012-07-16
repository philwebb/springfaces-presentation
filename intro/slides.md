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

* JSF becomes the V in MVC
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

<pre/>
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
<pre/>
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
<pre/>
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
<pre/>
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

<pre/>
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
<pre/>
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
<pre/>

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

<pre/>

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
					label="#{label}">
			</span>
			<ui:insert/>
		</div>
	</ui:composition>


<!SLIDE small>

# Example: #

	@@@ xml
	<ui:decorate template="/WEB-INF/layout/form.xhtml">
		<h:inputText id="firstName" label="First Name" 
			value="#{bean.firstName}"/>
		<ui:param name="label" value="First Name"/>
		<ui:param name="for" value="firstName"/>
	</ui:decorate>

	<ui:decorate template="/WEB-INF/layout/form.xhtml">
		<h:inputText id="lastName" label="Last Name" 
			value="#{bean.lastName}"/>
		<ui:param name="label" value="Last Name"/>
		<ui:param name="for" value="lastName"/>
	</ui:decorate>

	<!-- More form elements -->



<!SLIDE subsection>

# Internationalization #

<!SLIDE subsection>

# Exception Handling #

<!SLIDE subsection>

# Conversion and Validation #

